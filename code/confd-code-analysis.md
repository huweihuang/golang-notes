> confd的源码参考：https://github.com/kelseyhightower/confd

本文分析的`confd`的版本是`v0.16.0`，代码参考：https://github.com/kelseyhightower/confd/tree/v0.16.0。

# 1. [Main](https://github.com/kelseyhightower/confd/blob/v0.16.0/confd.go#L16)

confd的入口函数 Main 函数，先解析参数，如果是打印版本信息的参数，则执行打印版本的命令。

```go
func main() {
	flag.Parse()
	if config.PrintVersion {
		fmt.Printf("confd %s (Git SHA: %s, Go Version: %s)\n", Version, GitSHA, runtime.Version())
		os.Exit(0)
	}
    ...
}    
```

其中版本信息记录在https://github.com/kelseyhightower/confd/blob/v0.16.0/version.go#L3

```go
const Version = "0.16.0"
```

## 1.1. [initConfig](https://github.com/kelseyhightower/confd/blob/v0.16.0/config.go#L82)

初始化配置文件。

```go
if err := initConfig(); err != nil {
	log.Fatal(err.Error())
}
```

`initConfig`函数对基本的配置内容做初始化，当没有指定后端存储的时候，设置默认存储。

```go
// initConfig initializes the confd configuration by first setting defaults,
// then overriding settings from the confd config file, then overriding
// settings from environment variables, and finally overriding
// settings from flags set on the command line.
// It returns an error if any.
func initConfig() error {
	_, err := os.Stat(config.ConfigFile)
	if os.IsNotExist(err) {
		log.Debug("Skipping confd config file.")
	} else {
		log.Debug("Loading " + config.ConfigFile)
		configBytes, err := ioutil.ReadFile(config.ConfigFile)
		if err != nil {
			return err
		}

		_, err = toml.Decode(string(configBytes), &config)
		if err != nil {
			return err
		}
	}

	// Update config from environment variables.
	processEnv()

	if config.SecretKeyring != "" {
		kr, err := os.Open(config.SecretKeyring)
		if err != nil {
			log.Fatal(err.Error())
		}
		defer kr.Close()
		config.PGPPrivateKey, err = ioutil.ReadAll(kr)
		if err != nil {
			log.Fatal(err.Error())
		}
	}

	if config.LogLevel != "" {
		log.SetLevel(config.LogLevel)
	}

	if config.SRVDomain != "" && config.SRVRecord == "" {
		config.SRVRecord = fmt.Sprintf("_%s._tcp.%s.", config.Backend, config.SRVDomain)
	}

	// Update BackendNodes from SRV records.
	if config.Backend != "env" && config.SRVRecord != "" {
		log.Info("SRV record set to " + config.SRVRecord)
		srvNodes, err := getBackendNodesFromSRV(config.SRVRecord)
		if err != nil {
			return errors.New("Cannot get nodes from SRV records " + err.Error())
		}

		switch config.Backend {
		case "etcd":
			vsm := make([]string, len(srvNodes))
			for i, v := range srvNodes {
				vsm[i] = config.Scheme + "://" + v
			}
			srvNodes = vsm
		}

		config.BackendNodes = srvNodes
	}
	if len(config.BackendNodes) == 0 {
		switch config.Backend {
		case "consul":
			config.BackendNodes = []string{"127.0.0.1:8500"}
		case "etcd":
			peerstr := os.Getenv("ETCDCTL_PEERS")
			if len(peerstr) > 0 {
				config.BackendNodes = strings.Split(peerstr, ",")
			} else {
				config.BackendNodes = []string{"http://127.0.0.1:4001"}
			}
		case "etcdv3":
			config.BackendNodes = []string{"127.0.0.1:2379"}
		case "redis":
			config.BackendNodes = []string{"127.0.0.1:6379"}
		case "vault":
			config.BackendNodes = []string{"http://127.0.0.1:8200"}
		case "zookeeper":
			config.BackendNodes = []string{"127.0.0.1:2181"}
		}
	}
	// Initialize the storage client
	log.Info("Backend set to " + config.Backend)

	if config.Watch {
		unsupportedBackends := map[string]bool{
			"dynamodb": true,
			"ssm":      true,
		}

		if unsupportedBackends[config.Backend] {
			log.Info(fmt.Sprintf("Watch is not supported for backend %s. Exiting...", config.Backend))
			os.Exit(1)
		}
	}

	if config.Backend == "dynamodb" && config.Table == "" {
		return errors.New("No DynamoDB table configured")
	}
	config.ConfigDir = filepath.Join(config.ConfDir, "conf.d")
	config.TemplateDir = filepath.Join(config.ConfDir, "templates")
	return nil
}
```

## 1.2. storeClient

```go
log.Info("Starting confd")

storeClient, err := backends.New(config.BackendsConfig)
if err != nil {
	log.Fatal(err.Error())
}
```

根据配置文件中的存储后端类型构造一个存储后端的client，其中主要调用的函数为[backends.New(config.BackendsConfig)](https://github.com/kelseyhightower/confd/blob/v0.16.0/backends/client.go#L29)。

当没有设置存储后端时，默认为`etcd`。

```go
if config.Backend == "" {
	config.Backend = "etcd"
}
backendNodes := config.BackendNodes
```

当存储后端为`file`类型的处理。

```go
if config.Backend == "file" {
	log.Info("Backend source(s) set to " + strings.Join(config.YAMLFile, ", "))
} else {
	log.Info("Backend source(s) set to " + strings.Join(backendNodes, ", "))
}
```

最后再根据不同类型的存储后端，调用不同的存储后端构建函数，本文只分析`redis`类型的存储后端。

```go
switch config.Backend {
case "consul":
	return consul.New(config.BackendNodes, config.Scheme,
		config.ClientCert, config.ClientKey,
		config.ClientCaKeys,
		config.BasicAuth,
		config.Username,
		config.Password,
	)
case "etcd":
	// Create the etcd client upfront and use it for the life of the process.
	// The etcdClient is an http.Client and designed to be reused.
	return etcd.NewEtcdClient(backendNodes, config.ClientCert, config.ClientKey, config.ClientCaKeys, config.BasicAuth, config.Username, config.Password)
case "etcdv3":
	return etcdv3.NewEtcdClient(backendNodes, config.ClientCert, config.ClientKey, config.ClientCaKeys, config.BasicAuth, config.Username, config.Password)
case "zookeeper":
	return zookeeper.NewZookeeperClient(backendNodes)
case "rancher":
	return rancher.NewRancherClient(backendNodes)
case "redis":
	return redis.NewRedisClient(backendNodes, config.ClientKey, config.Separator)
case "env":
	return env.NewEnvClient()
case "file":
	return file.NewFileClient(config.YAMLFile, config.Filter)
case "vault":
	vaultConfig := map[string]string{
		"app-id":    config.AppID,
		"user-id":   config.UserID,
		"role-id":   config.RoleID,
		"secret-id": config.SecretID,
		"username":  config.Username,
		"password":  config.Password,
		"token":     config.AuthToken,
		"cert":      config.ClientCert,
		"key":       config.ClientKey,
		"caCert":    config.ClientCaKeys,
		"path":      config.Path,
	}
	return vault.New(backendNodes[0], config.AuthType, vaultConfig)
case "dynamodb":
	table := config.Table
	log.Info("DynamoDB table set to " + table)
	return dynamodb.NewDynamoDBClient(table)
case "ssm":
	return ssm.New()
}
return nil, errors.New("Invalid backend")
```

其中redis类型的存储后端调用了`NewRedisClient`方法来构造redis的client。

```go
case "redis":
	return redis.NewRedisClient(backendNodes, config.ClientKey, config.Separator)
```

其中涉及三个参数：

- `backendNodes`：redis的节点地址。
- `ClientKey`：redis的密码。
- `Separator`：查找redis键的分隔符，该参数只用在redis类型。

`NewRedisClient`函数方法如下：

```go
// NewRedisClient returns an *redis.Client with a connection to named machines.
// It returns an error if a connection to the cluster cannot be made.
func NewRedisClient(machines []string, password string, separator string) (*Client, error) {
	if separator == "" {
		separator = "/"
	}
	log.Debug(fmt.Sprintf("Redis Separator: %#v", separator))
	var err error
	clientWrapper := &Client{machines: machines, password: password, separator: separator, client: nil, pscChan: make(chan watchResponse), psc: redis.PubSubConn{Conn: nil} }
	clientWrapper.client, _, err = tryConnect(machines, password, true)
	return clientWrapper, err
}
```

## 1.3. processor

```go
stopChan := make(chan bool)
doneChan := make(chan bool)
errChan := make(chan error, 10)

var processor template.Processor
switch {
case config.Watch:
	processor = template.WatchProcessor(config.TemplateConfig, stopChan, doneChan, errChan)
default:
	processor = template.IntervalProcessor(config.TemplateConfig, stopChan, doneChan, errChan, config.Interval)
}

go processor.Process()
```

当开启`watch`参数的时候，则构造`WatchProcessor`，否则构造`IntervalProcessor`，最后起一个goroutine。

```go
go processor.Process()
```

这块的逻辑在本文第二部分析。

## 1.4. signalChan

```go
signalChan := make(chan os.Signal, 1)
signal.Notify(signalChan, syscall.SIGINT, syscall.SIGTERM)
for {
	select {
	case err := <-errChan:
		log.Error(err.Error())
	case s := <-signalChan:
		log.Info(fmt.Sprintf("Captured %v. Exiting...", s))
		close(doneChan)
	case <-doneChan:
		os.Exit(0)
	}
}
```

# 2. [Process](https://github.com/kelseyhightower/confd/blob/v0.16.0/resource/template/processor.go#L12)

```go
type Processor interface {
	Process()
}
```

`Processor`是一个接口类型，主要的实现体有：

- `intervalProcessor`：默认的实现体，即没有添加watch参数。
- `watchProcessor`：添加watch参数的实现体。

## 2.1. intervalProcessor

```go
type intervalProcessor struct {
	config   Config
	stopChan chan bool
	doneChan chan bool
	errChan  chan error
	interval int
}
```

intervalProcessor根据config内容和几个channel构造一个intervalProcessor。

```go
func IntervalProcessor(config Config, stopChan, doneChan chan bool, errChan chan error, interval int) Processor {
	return &intervalProcessor{config, stopChan, doneChan, errChan, interval}
}
```

### 2.1.1. intervalProcessor.Process

```go
func (p *intervalProcessor) Process() {
	defer close(p.doneChan)
	for {
		ts, err := getTemplateResources(p.config)
		if err != nil {
			log.Fatal(err.Error())
			break
		}
		process(ts)
		select {
		case <-p.stopChan:
			break
		case <-time.After(time.Duration(p.interval) * time.Second):
			continue
		}
	}
}
```

通过解析config内容获取`TemplateResources`，其中核心函数为`process(ts)`，然后执行`t.process()`，该函数中会调用`t.sync()`。`t.process()`的具体逻辑后文分析。

```go
func process(ts []*TemplateResource) error {
	var lastErr error
	for _, t := range ts {
		if err := t.process(); err != nil {
			log.Error(err.Error())
			lastErr = err
		}
	}
	return lastErr
}
```

## 2.2. watchProcessor

```
type watchProcessor struct {
	config   Config
	stopChan chan bool
	doneChan chan bool
	errChan  chan error
	wg       sync.WaitGroup
}
```

watchProcessor根据config内容和几个channel构造一个watchProcessor。

```go
func WatchProcessor(config Config, stopChan, doneChan chan bool, errChan chan error) Processor {
	var wg sync.WaitGroup
	return &watchProcessor{config, stopChan, doneChan, errChan, wg}
}
```

### 2.2.1. watchProcessor.Process

```go
func (p *watchProcessor) Process() {
	defer close(p.doneChan)
	ts, err := getTemplateResources(p.config)
	if err != nil {
		log.Fatal(err.Error())
		return
	}
	for _, t := range ts {
		t := t
		p.wg.Add(1)
		go p.monitorPrefix(t)
	}
	p.wg.Wait()
}
```

`watchProcessor.Process`方法实现了`Processor`接口中定义的方法，通过解析config内容获取`TemplateResources`，再遍历`TemplateResources`执行`monitorPrefix`，有多少个`TemplateResources`就运行多少个`monitorPrefix`的goroutine。

### 2.2.2. monitorPrefix

```go
func (p *watchProcessor) monitorPrefix(t *TemplateResource) {
	defer p.wg.Done()
	keys := util.AppendPrefix(t.Prefix, t.Keys)
	for {
		index, err := t.storeClient.WatchPrefix(t.Prefix, keys, t.lastIndex, p.stopChan)
		if err != nil {
			p.errChan <- err
			// Prevent backend errors from consuming all resources.
			time.Sleep(time.Second * 2)
			continue
		}
		t.lastIndex = index
		if err := t.process(); err != nil {
			p.errChan <- err
		}
	}
}
```

先对配置文件中的`prefix`和`keys`参数进行拼接。

```go
keys := util.AppendPrefix(t.Prefix, t.Keys)
```

`AppendPrefix`函数如下：

```go
func AppendPrefix(prefix string, keys []string) []string {
	s := make([]string, len(keys))
	for i, k := range keys {
		s[i] = path.Join(prefix, k)
	}
	return s
}
```

接着再执行`storeClient`的`WatchPrefix`方法，因为`storeClient`是一个接口，对应不同类型的存储后端，`WatchPrefix`的实现逻辑也不同，本文分析的存储类型为`redis`。

```go
index, err := t.storeClient.WatchPrefix(t.Prefix, keys, t.lastIndex, p.stopChan)
if err != nil {
	p.errChan <- err
	// Prevent backend errors from consuming all resources.
	time.Sleep(time.Second * 2)
	continue
}
```

`storeClient.WatchPrefix`主要是获取`lastIndex`的值，这个值在`t.process()`中使用。

```go
t.lastIndex = index
if err := t.process(); err != nil {
	p.errChan <- err
}
```

## 2.3. TemplateResource.process

无论是否加`watch`参数，即`intervalProcessor`和`watchProcessor`最终都会调用到`TemplateResource.process`这个函数，而这个函数中的核心函数为`t.sync()`。

```go
// process is a convenience function that wraps calls to the three main tasks
// required to keep local configuration files in sync. First we gather vars
// from the store, then we stage a candidate configuration file, and finally sync
// things up.
// It returns an error if any.
func (t *TemplateResource) process() error {
	if err := t.setFileMode(); err != nil {
		return err
	}
	if err := t.setVars(); err != nil {
		return err
	}
	if err := t.createStageFile(); err != nil {
		return err
	}
	if err := t.sync(); err != nil {
		return err
	}
	return nil
}
```

### 2.3.1. setFileMode

`setFileMode`设置文件的权限，如果没有在配置文件指定`mode`参数则默认为`0644`，否则根据配置文件中指定的`mode`来设置文件权限。

```go
// setFileMode sets the FileMode.
func (t *TemplateResource) setFileMode() error {
	if t.Mode == "" {
		if !util.IsFileExist(t.Dest) {
			t.FileMode = 0644
		} else {
			fi, err := os.Stat(t.Dest)
			if err != nil {
				return err
			}
			t.FileMode = fi.Mode()
		}
	} else {
		mode, err := strconv.ParseUint(t.Mode, 0, 32)
		if err != nil {
			return err
		}
		t.FileMode = os.FileMode(mode)
	}
	return nil
}
```

### 2.3.2. setVars

`setVars`将后端存储中最新的值拿出来暂存到内存中供后续进程使用。其中根据不同的后端，`storeClient.GetValues`的逻辑可能不同，但通过接口的方式可以让不同的存储后端实现不同的获取值的方法。

```go
// setVars sets the Vars for template resource.
func (t *TemplateResource) setVars() error {
	var err error
	log.Debug("Retrieving keys from store")
	log.Debug("Key prefix set to " + t.Prefix)

	result, err := t.storeClient.GetValues(util.AppendPrefix(t.Prefix, t.Keys))
	if err != nil {
		return err
	}
	log.Debug("Got the following map from store: %v", result)

	t.store.Purge()

	for k, v := range result {
		t.store.Set(path.Join("/", strings.TrimPrefix(k, t.Prefix)), v)
	}
	return nil
}
```

### 2.3.3. createStageFile

`createStageFile`通过`src`的`template`文件和最新内存中的变量数据生成`StageFile`，该文件在sync中和目标文件进行比较，看是否有修改。即`StageFile`实际上是根据后端存储生成的最新的配置文件，如果这份配置文件跟当前的配置文件不同，表明后端存储的数据被更新了需要重新生成一份新的配置文件。

```go
// createStageFile stages the src configuration file by processing the src
// template and setting the desired owner, group, and mode. It also sets the
// StageFile for the template resource.
// It returns an error if any.
func (t *TemplateResource) createStageFile() error {
	log.Debug("Using source template " + t.Src)

	if !util.IsFileExist(t.Src) {
		return errors.New("Missing template: " + t.Src)
	}

	log.Debug("Compiling source template " + t.Src)

	tmpl, err := template.New(filepath.Base(t.Src)).Funcs(t.funcMap).ParseFiles(t.Src)
	if err != nil {
		return fmt.Errorf("Unable to process template %s, %s", t.Src, err)
	}

	// create TempFile in Dest directory to avoid cross-filesystem issues
	temp, err := ioutil.TempFile(filepath.Dir(t.Dest), "."+filepath.Base(t.Dest))
	if err != nil {
		return err
	}

	if err = tmpl.Execute(temp, nil); err != nil {
		temp.Close()
		os.Remove(temp.Name())
		return err
	}
	defer temp.Close()

	// Set the owner, group, and mode on the stage file now to make it easier to
	// compare against the destination configuration file later.
	os.Chmod(temp.Name(), t.FileMode)
	os.Chown(temp.Name(), t.Uid, t.Gid)
	t.StageFile = temp
	return nil
}
```

### 2.3.4. sync

```go
if err := t.sync(); err != nil {
	return err
}
```

`t.sync()`是执行confd核心功能的函数，将配置文件通过模板的方式自动生成，并执行检查命令和reload命令。该部分逻辑在本文第三部分分析。

# 3. [sync](https://github.com/kelseyhightower/confd/blob/v0.16.0/resource/template/resource.go#L238)

`sync`通过比较源文件和目标文件的差别，如果不同则重新生成新的配置，当设置了`check_cmd`和`reload_cmd`的时候，会执行`check_cmd`指定的检查命令，如果都没有问题则执行`reload_cmd`中指定的reload命令。

## 3.1. IsConfigChanged

`IsConfigChanged`比较源文件和目标文件是否相等，其中比较内容包括：`Uid`、`Gid`、`Mode`、`Md5`。只要其中任意值不同则认为两个文件不同。

```go
// IsConfigChanged reports whether src and dest config files are equal.
// Two config files are equal when they have the same file contents and
// Unix permissions. The owner, group, and mode must match.
// It return false in other cases.
func IsConfigChanged(src, dest string) (bool, error) {
	if !IsFileExist(dest) {
		return true, nil
	}
	d, err := FileStat(dest)
	if err != nil {
		return true, err
	}
	s, err := FileStat(src)
	if err != nil {
		return true, err
	}
	if d.Uid != s.Uid {
		log.Info(fmt.Sprintf("%s has UID %d should be %d", dest, d.Uid, s.Uid))
	}
	if d.Gid != s.Gid {
		log.Info(fmt.Sprintf("%s has GID %d should be %d", dest, d.Gid, s.Gid))
	}
	if d.Mode != s.Mode {
		log.Info(fmt.Sprintf("%s has mode %s should be %s", dest, os.FileMode(d.Mode), os.FileMode(s.Mode)))
	}
	if d.Md5 != s.Md5 {
		log.Info(fmt.Sprintf("%s has md5sum %s should be %s", dest, d.Md5, s.Md5))
	}
	if d.Uid != s.Uid || d.Gid != s.Gid || d.Mode != s.Mode || d.Md5 != s.Md5 {
		return true, nil
	}
	return false, nil
}
```

如果文件发生改变则执行`check_cmd`命令（有配置的情况下），重新生成配置文件，并执行`reload_cmd`命令（有配置的情况下）。

```go
if ok {
	log.Info("Target config " + t.Dest + " out of sync")
	if !t.syncOnly && t.CheckCmd != "" {
		if err := t.check(); err != nil {
			return errors.New("Config check failed: " + err.Error())
		}
	}
	log.Debug("Overwriting target config " + t.Dest)
	err := os.Rename(staged, t.Dest)
	if err != nil {
		if strings.Contains(err.Error(), "device or resource busy") {
			log.Debug("Rename failed - target is likely a mount. Trying to write instead")
			// try to open the file and write to it
			var contents []byte
			var rerr error
			contents, rerr = ioutil.ReadFile(staged)
			if rerr != nil {
				return rerr
			}
			err := ioutil.WriteFile(t.Dest, contents, t.FileMode)
			// make sure owner and group match the temp file, in case the file was created with WriteFile
			os.Chown(t.Dest, t.Uid, t.Gid)
			if err != nil {
				return err
			}
		} else {
			return err
		}
	}
	if !t.syncOnly && t.ReloadCmd != "" {
		if err := t.reload(); err != nil {
			return err
		}
	}
	log.Info("Target config " + t.Dest + " has been updated")
} else {
	log.Debug("Target config " + t.Dest + " in sync")
}
```

## 3.2. check

`check`检查暂存的配置文件即`stageFile`，该文件是由最新的后端存储中的数据生成的。

```go
if !t.syncOnly && t.CheckCmd != "" {
	if err := t.check(); err != nil {
		return errors.New("Config check failed: " + err.Error())
	}
}
```

`t.check()`只是执行配置文件中`checkcmd`参数指定的命令而已，根据是否执行成功来返回报错。当`check`命令产生错误的是，则直接return报错，不再执行重新生成配置文件和``reload`的操作了。

```go
// check executes the check command to validate the staged config file. The
// command is modified so that any references to src template are substituted
// with a string representing the full path of the staged file. This allows the
// check to be run on the staged file before overwriting the destination config
// file.
// It returns nil if the check command returns 0 and there are no other errors.
func (t *TemplateResource) check() error {
	var cmdBuffer bytes.Buffer
	data := make(map[string]string)
	data["src"] = t.StageFile.Name()
	tmpl, err := template.New("checkcmd").Parse(t.CheckCmd)
	if err != nil {
		return err
	}
	if err := tmpl.Execute(&cmdBuffer, data); err != nil {
		return err
	}
	return runCommand(cmdBuffer.String())
}
```

`check`会通过模板解析的方式解析出`checkcmd`中的`{{.src}}`部分，并用`stageFile`来替代。即check的命令是拉取最新后端存储的数据形成临时配置文件（stageFile），并通过指定的`checkcmd`来检查最新的临时配置文件是否合法，如果合法则替换会新的配置文件，否则返回错误。

## 3.3. Overwriting 

将`staged`文件命名为`Dest`文件的名字，读取`staged`文件中的内容并将它写入到`Dest`文件中，该过程实际上就是重新生成一份新的配置文件。`staged`文件的生成逻辑在函数`createStageFile`中。

```go
log.Debug("Overwriting target config " + t.Dest)
err := os.Rename(staged, t.Dest)
if err != nil {
	if strings.Contains(err.Error(), "device or resource busy") {
		log.Debug("Rename failed - target is likely a mount. Trying to write instead")
		// try to open the file and write to it
		var contents []byte
		var rerr error
		contents, rerr = ioutil.ReadFile(staged)
		if rerr != nil {
			return rerr
		}
		err := ioutil.WriteFile(t.Dest, contents, t.FileMode)
		// make sure owner and group match the temp file, in case the file was created with WriteFile
		os.Chown(t.Dest, t.Uid, t.Gid)
		if err != nil {
			return err
		}
	} else {
		return err
	}
}
```

## 3.4. reload

如果没有指定syncOnly参数并且指定了`ReloadCmd`则执行`reload`操作。

```go
if !t.syncOnly && t.ReloadCmd != "" {
	if err := t.reload(); err != nil {
		return err
	}
}
```

其中`t.reload()`实现如下：

```go
// reload executes the reload command.
// It returns nil if the reload command returns 0.
func (t *TemplateResource) reload() error {
	return runCommand(t.ReloadCmd)
}
```

`t.reload()`和`t.check()`都调用了`runCommand`函数：

```go
// runCommand is a shared function used by check and reload
// to run the given command and log its output.
// It returns nil if the given cmd returns 0.
// The command can be run on unix and windows.
func runCommand(cmd string) error {
	log.Debug("Running " + cmd)
	var c *exec.Cmd
	if runtime.GOOS == "windows" {
		c = exec.Command("cmd", "/C", cmd)
	} else {
		c = exec.Command("/bin/sh", "-c", cmd)
	}

	output, err := c.CombinedOutput()
	if err != nil {
		log.Error(fmt.Sprintf("%q", string(output)))
		return err
	}
	log.Debug(fmt.Sprintf("%q", string(output)))
	return nil
}
```

# 4. [redisClient.WatchPrefix](https://github.com/kelseyhightower/confd/blob/v0.16.0/backends/redis/client.go#L228)

`redisClient.WatchPrefix`是当用户设置了`watch`参数的时候，并且存储后端为`redis`，则会调用到redis的watch机制。其中`redisClient.WatchPrefix`是redis存储类型的时候实现了`StoreClient`接口的`WatchPrefix`方法。

```go
// The StoreClient interface is implemented by objects that can retrieve
// key/value pairs from a backend store.
type StoreClient interface {
	GetValues(keys []string) (map[string]string, error)
	WatchPrefix(prefix string, keys []string, waitIndex uint64, stopChan chan bool) (uint64, error)
}
```

`StoreClient`是对后端存储类型的抽象，常用的后端存储类型有`Etcd`和`Redis`等，不同的后端存储类型`GetValues`和`WatchPrefix`的具体实现不同，本文主要分析Redis类型的`watch`机制。

## 4.1. WatchPrefix

WatchPrefix的调用函数在[monitorPrefix](#222-monitorPrefix)的部分，具体参考：

```go
func (p *watchProcessor) monitorPrefix(t *TemplateResource) {
	defer p.wg.Done()
	keys := util.AppendPrefix(t.Prefix, t.Keys)
	for {
		index, err := t.storeClient.WatchPrefix(t.Prefix, keys, t.lastIndex, p.stopChan)
		if err != nil {
			p.errChan <- err
			// Prevent backend errors from consuming all resources.
			time.Sleep(time.Second * 2)
			continue
		}
		t.lastIndex = index
		if err := t.process(); err != nil {
			p.errChan <- err
		}
	}
}
```

redis的`watch`主要通过`pub-sub`的机制，即WatchPrefix会根据传入的`prefix`起一个sub的监听机制，而在写入redis的数据的同时需要执行redis的publish操作，channel为符合prefix的值，value为给定命令之一，实际上是给定命令之一，具体是什么命令并没有关系，则会触发watch机制，从而自动更新配置，给定的命令列表如下：

```go
"del", "append", "rename_from", "rename_to", "expire", "set", "incrby", "incrbyfloat", "hset", "hincrby", "hincrbyfloat", "hdel"
```

sub监听的key的格式如下：

```go
__keyspace@0__:{prefix}/*
```

如果只是写入redis数据而没有自动执行publish的操作，并不会触发redis的watch机制来自动更新配置。但是如果使用etcd，则etcd的watch机制，只需要用户写入或更新数据就可以自动触发更新配置。

`WatchPrefix`源码如下：

```go
func (c *Client) WatchPrefix(prefix string, keys []string, waitIndex uint64, stopChan chan bool) (uint64, error) {
		
	if waitIndex == 0 {
		return 1, nil
	}

	if len(c.pscChan) > 0 {
		var respChan watchResponse
		for len(c.pscChan) > 0 {
			respChan = <-c.pscChan
		}
		return respChan.waitIndex, respChan.err
	}

	go func() {
		if c.psc.Conn == nil {
			rClient, db, err := tryConnect(c.machines, c.password, false);
	
			if err != nil {
				c.psc = redis.PubSubConn{Conn: nil}
				c.pscChan <- watchResponse{0, err}
				return
			}
		
			c.psc = redis.PubSubConn{Conn: rClient}		

			go func() {
				defer func() {
					c.psc.Close()
					c.psc = redis.PubSubConn{Conn: nil}
				}()
				for {
					switch n := c.psc.Receive().(type) {
						case redis.PMessage:
							log.Debug(fmt.Sprintf("Redis Message: %s %s\n", n.Channel, n.Data))
							data := string(n.Data)
							commands := [12]string{"del", "append", "rename_from", "rename_to", "expire", "set", "incrby", "incrbyfloat", "hset", "hincrby", "hincrbyfloat", "hdel"}
							for _, command := range commands {
								if command == data {
									c.pscChan <- watchResponse{1, nil}
									break
								}
							}
						case redis.Subscription:
							log.Debug(fmt.Sprintf("Redis Subscription: %s %s %d\n", n.Kind, n.Channel, n.Count))
							if n.Count == 0 {
								c.pscChan <- watchResponse{0, nil}
								return
							}
						case error:
							log.Debug(fmt.Sprintf("Redis error: %v\n", n))
							c.pscChan <- watchResponse{0, n}
							return
					}
				}
			}()
			
			c.psc.PSubscribe("__keyspace@" + strconv.Itoa(db) + "__:" + c.transform(prefix) + "*")
		}
	}()

	select {
	case <-stopChan:
		c.psc.PUnsubscribe()
		return waitIndex, nil
	case r := <- c.pscChan:
		return r.waitIndex, r.err
	}
}
```

# 5. 总结

1. confd的作用是通过将配置存放到存储后端，来自动触发更新配置的功能，其中常用的后端有`Etcd`和`Redis`等。
2. 不同的存储后端，watch机制不同，例如Etcd只需要更新key便可以触发自动更新配置的操作，而redis除了更新key还需要执行`publish`的操作。
3. 可以通过配置`check_cmd`来校验配置文件是否正确，如果配置文件非法则不会执行自动更新配置和`reload`的操作，但是当存储后端存入的非法数据，会导致每次校验都是失败的，即使后面新增的配置部分是合法的，所以需要有机制来控制存入存储后端的数据始终是合法的。



参考：

- https://github.com/kelseyhightower/confd/tree/v0.16.0

