# 1212300229_p3
周奎君，1212300229，Unix系统分析作业3

###1 cdev结构体
 在Linux2.6 内核中，使用cdev结构体来描述一个字符设备，cdev结构体的定义如下：

```
struct cdev {

      struct kobject kobj;

      struct module *owner;  /*通常为THIS_MODULE*/

      struct file_operations *ops; /*在cdev_init()这个函数里面与cdev结构联系起来*/

      struct  list_head list;

      dev_t  dev;  /*设备号*/

      unsigned int count;

}
```
 cdev 结构体的dev_t 成员定义了设备号，为32位，其中12位是主设备号，20位是次设备号，我们只需使用二个简单的宏就可以从dev_t 中获取主设备号和次设备号：

MAJOR(dev_t dev)

MINOR(dev_t dev)

相反地，可以通过主次设备号来生成dev_t：

MKDEV(int major,int minor)


###2、字符设备驱动模块加载与卸载函数
 在字符设备驱动模块加载函数中应该实现设备号的申请和cdev 结构的注册，而在卸载函数中应该实现设备号的释放与cdev结构的注销。

  我们一般习惯将cdev内嵌到另外一个设备相关的结构体里面，该设备包含所涉及的cdev、私有数据及信号量等等信息。常见的设备结构体、模块加载函数、模块卸载函数形式如下：

  ```
/*设备结构体*/

struct  xxx_dev{

      struct   cdev   cdev;

      char *data;

      struct semaphore sem;

      ......

};

/*模块加载函数*/

static int   __init  xxx_init(void)

{
      .......

      初始化cdev结构;

      申请设备号；

      注册设备号；
  
       申请分配设备结构体的内存；  /*非必须*/

}

 
/*模块卸载函数*/

static void  __exit   xxx_exit(void)

{

       .......

       释放原先申请的设备号；

       释放原先申请的内存；

       注销cdev设备；

}
  ```
###3、字符设备驱动的 file_operations 结构体重成员函数
```
/*读设备*/
ssize_t   xxx_read(struct file *filp,  char __user *buf,  size_t  count,  loff_t *f_pos)
{
        ......
        使用filp->private_data获取设备结构体指针；
        分析和获取有效的长度；
        /*内核空间到用户空间的数据传递*/
        copy_to_user(void __user *to,  const void *from,  unsigned long count);
        ......
}

/*写设备*/

ssize_t   xxx_write(struct file *filp,  const char  __user *buf,  size_t  count,  loff_t *f_pos)
{
        ......
        使用filp->private_data获取设备结构体指针；
        分析和获取有效的长度；
        /*用户空间到内核空间的数据传递*/
        copy_from_user(void *to,  const  void   __user *from,  unsigned long count);
        ......
}

/*ioctl函数*/

static int xxx_ioctl(struct inode *inode,struct file *filp,unsigned int cmd,unsigned long arg)
{
      ......
      switch(cmd){
           case  xxx_CMD1:
                        ......
                        break;
           case  xxx_CMD2:
                       .......
                      break;
           default:
                      return -ENOTTY;  /*不能支持的命令*/
      }
      return 0;

}
```
###4、字符设备驱动文件操作结构体模板
```
struct file_operations xxx_fops = {
      .owner = THIS_MODULE,
      .open = xxx_open,
      .read = xxx_read,
     .write = xxx_write,
     .close = xxx_release,
     .ioctl = xxx_ioctl,
     .lseek = xxx_llseek,
};
```
上面的写法需要注意二点，一：结构体成员之间是以逗号分开的而不是分号，结构体字段结束时最后应加上分号。

###参考博客：[http://blog.csdn.net/lwj103862095/article/details/8539522](http://blog.csdn.net/lwj103862095/article/details/8539522)