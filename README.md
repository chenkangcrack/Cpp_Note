# 单例

local static对象在c++11之前不是线程安全的。如果一个线程在构造一半时另外一个线程进来了，就会重复构造进而产生内存泄漏（为何重复构造会产生内存泄露？）c++11规定，在一个线程开始local static对象构造的过程时，另外一个线程进来时会等待，直到前一个线程构造完毕。

#include <mutex>

class Singleton {
private:
	Singleton() {};
	~Singleton() {};
	Singleton(const Singleton&);
	Singleton& operator=(const Singleton&);
public:
	static Singleton& getInstance() {
		static Singleton s;
		return s;
	}
};

//带锁的单例
//每次获取单例都得加锁，访问变成串行
std::mutex m;
template<class T>
T& getSingleton() {
	std::unique_lock lock(m);
	static T instance;
	return instance;
}

// Double check lock
class Singleton {
private:
	Singleton() {};
	Singleton(const Singleton&) {};
public:
	static Singleton& getInstance();
private:
	static std::mutex m;
	static Singleton* instance;
};

Singleton* Singleton::instance = nullptr;
Singleton& Singleton::getInstance() {
	if (instance == nullptr) {
		std::unique_lock<std::mutex> lock(m);
		if (instance == nullptr) {
			instance = new Singleton();
		}
	}
	return *instance;
}



//内存屏障

//在某些内存模型中有可能出现instance不为nullptr但是没有构造完成得情况，此时的锁其实并没有起作用
//https://blog.csdn.net/qq_35280514/article/details/70211845
//此时只需把instance类型修改一下
static std::atomic<Singleton *> m_instance(nullptr);