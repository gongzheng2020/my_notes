
---

本文主要介绍如何创建一个简单的字符设备驱动，并编写应用程序进行调用，可在x86电脑上测试。

字符设备驱动程序是以内核模块的形式存在的， 因此我们根据内核模块框架编写驱动，并向系统注册一个新的字符设备。主要需要这几样东西：字符设备结构体cdev，设备编号devno， 以及最重要的操作方式结构体file_operations。

字符驱动程序如下：
chrdev.c
```C
#include <linux/init.h>
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/uaccess.h>

#define DEV_NAME "EmbedCharDev"
#define DEV_CNT (1)
#define BUFF_SIZE 128

//定义字符设备的设备号
static dev_t devno;

//定义字符设备结构体chr_dev
static struct cdev chr_dev;

//数据缓冲区
static char vbuf[BUFF_SIZE];

static int chr_dev_open(struct inode *inode, struct file *filp);
static int chr_dev_release(struct inode *inode, struct file *filp);
static ssize_t chr_dev_write(struct file *filp, const char __user * buf, size_t count, loff_t *ppos);
static ssize_t chr_dev_read(struct file *filp, char __user * buf, size_t count, loff_t *ppos);

static struct file_operations chr_dev_fops =
{
	.owner = THIS_MODULE,	
	.open = chr_dev_open,	
	.release = chr_dev_release,	
	.write = chr_dev_write,
	.read = chr_dev_read,
};

static int chr_dev_open(struct inode *inode, struct file *filp)
{
	printk("\nopen\n");
	return 0;
}

static int chr_dev_release(struct inode *inode, struct file *filp)
{
	printk("\nrelease\n");
	return 0;
}

static ssize_t chr_dev_write(struct file *filp, const char __user * buf, size_t count, loff_t *ppos)
{
	unsigned long p = *ppos;
	int ret;
	int tmp = count ;
	if(p > BUFF_SIZE)
		return 0;
	if(tmp > BUFF_SIZE - p)
		tmp = BUFF_SIZE - p;
	ret = copy_from_user(vbuf, buf, tmp);
	*ppos += tmp;
	return tmp;
}

static ssize_t chr_dev_read(struct file *filp, char __user * buf, size_t count, loff_t *ppos)
{
	unsigned long p = *ppos;
	int ret;
	int tmp = count ;
	static int i = 0;
	i++;
	if(p >= BUFF_SIZE)
		return 0;
	if(tmp > BUFF_SIZE - p)
		tmp = BUFF_SIZE - p;
	ret = copy_to_user(buf, vbuf+p, tmp);
	*ppos +=tmp;
	return tmp;
}

static int __init chrdev_init(void)
{
	int ret = 0;
	printk("chrdev init\n");
	//第一步
	//采用动态分配的方式，获取设备编号，次设备号为0，
	//设备名称为EmbedCharDev，可通过命令cat /proc/devices查看
	//DEV_CNT为1，当前只申请一个设备编号
	ret = alloc_chrdev_region(&devno, 0, DEV_CNT, DEV_NAME);
	if(ret < 0){
		printk("fail to alloc devno\n");
		goto alloc_err;
	}
	
	//第二步
	//关联字符设备结构体cdev与文件操作结构体file_operations
	cdev_init(&chr_dev, &chr_dev_fops);
	
	//第三步
	//添加设备至cdev_map散列表中
	ret = cdev_add(&chr_dev, devno, DEV_CNT);
	if(ret < 0)
	{	
		printk("fail to add cdev\n");	
		goto add_err;	
	}
	return 0;
	
add_err:
	//添加设备失败时，需要注销设备号
	unregister_chrdev_region(devno, DEV_CNT);
alloc_err:
	return ret;
}

module_init(chrdev_init);

static void __exit chrdev_exit(void)
{
	printk("chrdev exit\n");
	unregister_chrdev_region(devno, DEV_CNT);
	cdev_del(&chr_dev);
}

module_exit(chrdev_exit);

MODULE_LICENSE("GPL");
```

创建并加载完驱动后，我们还可以编写应用程序来对该设备进行调用，程序如下：

```C
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <string.h>

char *wbuf = "Hello World\n";
char rbuf[128];

int main(void)
{
    printf("EmbedCharDev test\n");
    //打开文件
    int fd = open("/dev/chrdev", O_RDWR);
    //写入数据
    write(fd, wbuf, strlen(wbuf));
    //写入完毕，关闭文件
    close(fd);
    //打开文件
     fd = open("/dev/chrdev", O_RDWR);
    //读取文件内容
    read(fd, rbuf, 128);
    //打印读取的内容
    printf("The content : %s", rbuf);
    //读取完毕，关闭文件
    close(fd);
    return 0;
}
```

驱动将会被编译为`.ko`文件，应用程序会被编译为可执行文件，具体的makefile文件如下：
main.c
``` C
# 本机(x86)编译：使用当前运行内核的构建目录
KERNEL_DIR=/lib/modules/$(shell uname -r)/build

obj-m := chrdev.o
out =  chrdev_test

all:
	$(MAKE) -C $(KERNEL_DIR) M=$(CURDIR) modules
	$(CROSS_COMPILE)gcc -o $(out) main.c

.PHONY:clean
clean:
	$(MAKE) -C $(KERNEL_DIR) M=$(CURDIR) clean
	rm $(out)
```

执行`sudo insmod chrdev.ko`命令后可以看到模块已经注册成功了：

![[Pasted image 20260822191125.png]]

再使用`sudo mknod /dev/chrdev c 504 0`创建设备，可以在`/dev`目录中查看到：

![[Pasted image 20260822191903.png]]

运行我们自己编写的应用程序，可以查看到效果：

![[Pasted image 20260822192013.png]]

或者可以使用echo或者cat命令来进行测试：

```C
sudo sh -c "echo 'EmbedCharDev test' > /dev/chrdev"
cat /dev/chrdev
```

![[Pasted image 20260822192205.png]]

当我们不需要该内核模块的时候，我们可以执行以下命令：

```C
sudo rmmod chrdev.ko
sudo rm /dev/chrdev
```
# 参考文献

1.[4.6 字符设备驱动程序实验](https://doc.embedfire.com/linux/imx6/driver/zh/latest/linux_driver/character_device.html#id14)