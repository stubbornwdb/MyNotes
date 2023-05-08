高内聚低耦合的情况下：线程  操作  资源类

判断/干活/通知   

防止虚假唤醒

1.1先创建资源类

暴露出接口供线程使用

```
class Ticket{
	private int number = 30;
	//List list = new ArrayList();
	Lock lock = new ReentrantLock();
	public void sale(){
		//trylock快捷
		lock.lock();
		try{
			if(number>0){
				//mycurr快捷键
				System.out.println();
			}
		}catch(Exception e){
			//....
		}finally{
			lock.unlock();
		}
	}
}

public class SaleTicketDemo1{
	public static void main(String[] args){
		Ticket ticket = new Ticket();
		//快捷键 newThread
		new Thread(new Runnnable(){
			@Override
			public void run(){
				for(int i=1;i<=40;i++){
					ticket.sale();
				}
			}
		},"A").start();
		
		new Thread(new Runnnable(){
			@Override
			public void run(){
				for(int i=1;i<=40;i++){
					ticket.sale();
				}
			}
		},"B").start();
		
		new Thread(new Runnnable(){
			@Override
			public void run(){
				for(int i=1;i<=40;i++){
					ticket.sale();
				}
			}
		},"C").start();
		
		//改成Lamda表达式
		new Thread(()->{for(int i=1;i<=40;i++) ticket.sale();}, "D").start();
	}
}
```

Lamda表达式：解决匿名内部类冗余

函数式编程

1.拷贝小括号，写死右箭头，落地大括号

2.@FunctionalInterface

3.default  方法（可多个）静态方法（也可多个）

面向接口编程  函数式接口

@FunctionalInterface

interface Foo{

​	//仅有一个方法（不会搞错，直接省略）

​	piblic void sayHello();

}





----

线程不安全？ArrayList    Set  Map

1.故障现象 

java.util.ConcurrentModificationException

2.导致原因

多个线程对同一个资源类、集合类进行读写，导致读写不一致的问题出现

3.解决方法

​	3.1 vector (给ArrayList加锁)

​	3.2 Collections.snchronizedList(new ArrayList<>());

​	3.3 CopyOnWriteArrayList() 写时复制   CopyOnWriteArraySet()  ConcurrentHashMap()

4.优化建议（同样的错误不犯两次）

