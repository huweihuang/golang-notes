# 2. client-go中接口的使用分析

以下以`k8s.io/client-go/kubernetes/typed/core/v1/pod.go`的pod对象做分析。

## 2.1 接口设计与定义

## 2.1.1 接口组合

```go
// PodsGetter has a method to return a PodInterface.
// A group's client should implement this interface.
type PodsGetter interface {
	Pods(namespace string) PodInterface
}
```

## 2.1.2 接口定义 {#index1}

```go
// PodInterface has methods to work with Pod resources.
type PodInterface interface {
	Create(*v1.Pod) (*v1.Pod, error)
	Update(*v1.Pod) (*v1.Pod, error)
	UpdateStatus(*v1.Pod) (*v1.Pod, error)
	Delete(name string, options *meta_v1.DeleteOptions) error
	DeleteCollection(options *meta_v1.DeleteOptions, listOptions meta_v1.ListOptions) error
	Get(name string, options meta_v1.GetOptions) (*v1.Pod, error)
	List(opts meta_v1.ListOptions) (*v1.PodList, error)
	Watch(opts meta_v1.ListOptions) (watch.Interface, error)
	Patch(name string, pt types.PatchType, data []byte, subresources ...string) (result *v1.Pod, err error)
	PodExpansion
}
```

`PodInterface`接口定义了pod对象所使用的方法，一般为增删改查等。其他kubernetes资源对象的接口定义类似，区别在于入参和出参与对象相关。例如`Create(*v1.Pod) (*v1.Pod, error)`方法定义的入参出参为`*v1.Pod`。如果要实现该接口，即实现该接口的所有方法。

## 2.2 接口的实现 {#index2}

## 2.2.1 结构体的定义

```go
// pods implements PodInterface
type pods struct {
	client rest.Interface
	ns     string
}
```

## 2.2.2 new函数[构造函数]

```go
// newPods returns a Pods
func newPods(c *CoreV1Client, namespace string) *pods {
	return &pods{
		client: c.RESTClient(),
		ns:     namespace,
	}
}
```

## 2.2.3 方法的实现

**Get**

```go
// Get takes name of the pod, and returns the corresponding pod object, and an error if there is any.
func (c *pods) Get(name string, options meta_v1.GetOptions) (result *v1.Pod, err error) {
	result = &v1.Pod{}
	err = c.client.Get().
		Namespace(c.ns).
		Resource("pods").
		Name(name).
		VersionedParams(&options, scheme.ParameterCodec).
		Do().
		Into(result)
	return
}
```

**List**

```go
// List takes label and field selectors, and returns the list of Pods that match those selectors.
func (c *pods) List(opts meta_v1.ListOptions) (result *v1.PodList, err error) {
	result = &v1.PodList{}
	err = c.client.Get().
		Namespace(c.ns).
		Resource("pods").
		VersionedParams(&opts, scheme.ParameterCodec).
		Do().
		Into(result)
	return
}
```

**Create**

```go
// Create takes the representation of a pod and creates it.  Returns the server's representation of the pod, and an error, if there is any.
func (c *pods) Create(pod *v1.Pod) (result *v1.Pod, err error) {
	result = &v1.Pod{}
	err = c.client.Post().
		Namespace(c.ns).
		Resource("pods").
		Body(pod).
		Do().
		Into(result)
	return
}
```

**Update**

```go
// Update takes the representation of a pod and updates it. Returns the server's representation of the pod, and an error, if there is any.
func (c *pods) Update(pod *v1.Pod) (result *v1.Pod, err error) {
	result = &v1.Pod{}
	err = c.client.Put().
		Namespace(c.ns).
		Resource("pods").
		Name(pod.Name).
		Body(pod).
		Do().
		Into(result)
	return
}
```

**Delete**

```go
// Delete takes name of the pod and deletes it. Returns an error if one occurs.
func (c *pods) Delete(name string, options *meta_v1.DeleteOptions) error {
	return c.client.Delete().
		Namespace(c.ns).
		Resource("pods").
		Name(name).
		Body(options).
		Do().
		Error()
}
```

## 2.3 接口的调用

示例：

```go
// 创建clientset实例
clientset, err := kubernetes.NewForConfig(config)
// 具体的调用
pods, err := clientset.CoreV1().Pods("").List(metav1.ListOptions{})
```

clientset实现了接口`Interface`，`Interface`是个接口组合，包含各个client的接口类型。例如`CoreV1()`方法对应的接口类型是`CoreV1Interface`。

以下是clientset的`CoreV1()`方法实现：

```go
// CoreV1 retrieves the CoreV1Client
func (c *Clientset) CoreV1() corev1.CoreV1Interface {
	return c.coreV1
}
```

该方法可以理解为是一个`构造函数`。构造函数的返回值类型是一个接口类型`CoreV1Interface`，而return的返回值是实现了该接口类型的结构体对象`c.coreV1`。

**接口类型是一种特殊的类型，接口类型与结构体对象之间的关系好比变量类型与变量之间的关系**。其中的结构体对象必须实现了该接口类型的所有方法。

所以clientset的`CoreV1()`方法实现是返回一个`CoreV1Client`结构体对象。该结构体对象实现了`CoreV1Interface`接口，该接口也是一个接口组合。

```go
type CoreV1Interface interface {
	RESTClient() rest.Interface
	ComponentStatusesGetter
	ConfigMapsGetter
	EndpointsGetter
	EventsGetter
	LimitRangesGetter
	NamespacesGetter
	NodesGetter
	PersistentVolumesGetter
	PersistentVolumeClaimsGetter
	PodsGetter
	PodTemplatesGetter
	ReplicationControllersGetter
	ResourceQuotasGetter
	SecretsGetter
	ServicesGetter
	ServiceAccountsGetter
}
```

而实现的`Pods()`方法是其中的`PodsGetter`接口。

`Pods()`同`CoreV1()`一样是个构造函数，构造函数的返回值类型是`PodInterface`接口，返回值是实现了`PodInterface`接口的`pods`结构体对象。

```go
func (c *CoreV1Client) Pods(namespace string) PodInterface {
	return newPods(c, namespace)
}
```

而`PodInterface`接口定义参考[接口定义](#index1)，`pods`对象实现了`PodInterface`接口的方法，具体参考[接口的实现](#index2)。

最终调用了`pods`对象的`List()`方法。

```go
pods, err := clientset.CoreV1().Pods("").List(metav1.ListOptions{})
```

即以上代码就是不断调用实现了某接口的结构体对象的构造函数，生成具体的结构体对象，再调用结构体对象的某个具体方法。
