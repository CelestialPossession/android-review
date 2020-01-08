##### Glide.with(Context)->RequestManager

1、根据传入的上下文，查找对应上下文中的RequestManagerRetriever，再查找对应的RequestManager。

2、->RequestManagerRetriever.get(context)->获取FragmentManager->RequestManagerRetriever.supportFragmentGet()。从FramgentManger中获取SupportRequestManagerFragment。从SupportRequestManagerFragment获取RequestManager，如果没有找到，就创建一个，并绑定。

##### RequestManager

1、在RequestManager的构造方法中，RequestManager将自己加入SupportRequestManagerFragment的生命周期监听。

2、将自身加入到Glide的RequestManager列表

3、负责创建请求相关的RequestBuilder，并管理所有的Request

##### SupportRequestManagerFragment

1、在相应的FragmentManager中加入SupportRequestManagerFragment，并利用Fragment的生命周期来回调AndoirdLifeCycler中的生命钩子方法，从而控制RequestManager的请求

***

##### RequestManager.load()->RequestBuilder

1、asDrawable()：创建一个RequestBuilder对象，并返回

2、load()：绑定请求数据，并返回



***

##### Glide.apply(RequestOptions)->RequestBuilder

1、调用的是RequestBuilder.apply(RequestOptions)





***

##### Glide.into(View)

1、调用的是RequestBuilder.into(View)



##### RequestBuilder

1、into(View)中，执行下载方法