# 3. 通用接口设计

## 3.1 接口定义

```go
// ProjectManager manage life cycle of Deployment and Resources
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

## 3.2 结构体定义

```go
// pods implements PodInterface
type pods struct {
	client rest.Interface
	ns     string
}
```

## 3.3 构造函数

```go
// newPods returns a Pods
func newPods(c *CoreV1Client, namespace string) *pods {
	return &pods{
		client: c.RESTClient(),
		ns:     namespace,
	}
}
```

## 3.4 结构体实现

**List()**

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

## 3.5 接口调用

```go
pods, err := clientset.CoreV1().Pods("").List(metav1.ListOptions{})
```

## 3.6 其他接口设计示例

```go
type XxxManager interface {
	Create(args argsType) (*XxxStruct, error)
	Get(args argsType) (**XxxStruct, error)
	Update(args argsType) (*XxxStruct, error)
	Delete(name string, options *DeleleOptions) error
}

type XxxManagerImpl struct {
	Name string
	Namespace string
	kubeCli *kubernetes.Clientset
}

func NewXxxManagerImpl (namespace, name string, kubeCli *kubernetes.Clientset) XxxManager {
	return &XxxManagerImpl{
		Name name,
		Namespace namespace,
		kubeCli: kubeCli,
	}
}

func (xm *XxxManagerImpl) Create(args argsType) (*XxxStruct, error) {
	//具体的方法实现
}
```

