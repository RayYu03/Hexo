---
title: linux reviews
date: 2016-10-30 22:35:17
tags: 'linux'
---
## Makefile:
```c
// main.c
#include "mytool1.h"
#include "mytool2.h"

int main(int argc, char const *argv[]) {
  mytool1_print("hello tool 1.");
  mytool2_print("hello tool 2.");
  return 0;
}
// mytool1.c
#include "mytool1.h"
#include <stdio.h>
void mytool1_print(char *str) {
  printf("This is mytool1 print : %s\n", str);
}

// mytool1.h
void mytool1_print(char *str);

// mytool2.c
#include "mytool2.h"
#include <stdio.h>
void mytool2_print(char *str) {
  printf("This is mytool2 print : %s\n", str);
}

// mytool2.h
void mytool2_print(char *str);
```
```Makefile
// Makefile
CC = gcc
OBJ = main.o mytool1.o mytool2.o
main : $(OBJ)
	$(CC) -o main $(OBJ)
main.o : main.c mytool1.h mytool2.h
mytool1.o : mytool1.c mytool1.h
mytool2.o : mytool2.c mytool2.h
.PHONY : clean
clean :
	rm -rf $(OBJ) main
```
<!-- more -->
## 进程
```c
// fork.c
#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

int main(){
  pid_t res;
  res = fork();
  printf("%d\n",res);
  if (res == -1){
    // fork失败
    perror("fork fail.");
    exit(1);
  }
  else if (res == 0){
    // 子进程
    printf("child : PID = %d, PPID = %d\n", getpid(), getppid());
  }
  else{
    // 父进程
    printf("parent : PID = %d, PPID = %d\n", getpid(), getppid());
  }
  return 0;
}

```
```c
// waitpid.c
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

int main(){

  pid_t res1, res2;

  res1 = fork();

  if (res1 == -1){
    perror("fork fail");
    exit(1);
  }
  else if (res1 == 0){
    printf("sleep 5s in child.\n");
    sleep(5);
    _exit(0);
  }
  else{
    do {
        // 非轮询
        res2 = waitpid(res1, NULL, WNOHANG);
        if (res2 == 0){
          printf("The child process has not exited.\n");
          sleep(1);
        }
    } while(res2 == 0);

    if (res1 == res2){
      printf("The child process has exited.\n");
    }
  }
  return 0;
}
```
```c
// exit.c
#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

int main(){

  pid_t res;
  res = fork();

  if (res == -1){
    perror("fork fail");
    exit(1);
  }
  else if (res == 0){
    printf("test _exit() \n");
    printf("This is a content in buffer.");
    _exit(0);
  }
  else{
    printf("test exit() \n");
    printf("This is a content in buffer.\n");
    exit(0);
  }
  return 0;
}

```
```c
// exec族
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
int main(){
  pid_t res;
  res = fork();
  if (res == 0) {
    // execlp("ls","ls","-l",NULL)
    if (execl("/bin/pwd", "pwd", NULL) < 0) {
      perror("execlp error.");
    }
  }
}

```
## 线程
```c
// thread1.c
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>
#include <unistd.h>

void thread (void){
  int i;
  for (i = 0; i < 3; i++) {
    printf("This is a pthread process.\n");
  }
}

int main(void) {
  int i, ret;
  pthread_t id;

  ret = pthread_create(&id, NULL, (void*) thread, NULL);
  if (ret != 0){
    printf("Create pthread error.\n");
    exit(0);
  }
  for (i = 0; i < 3; i++) {
    printf("This is the main process.\n");
  }
  pthread_join(id, NULL);
  return 0;
}
```
```c
// thread2.c
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>
#include <unistd.h>

void thread (void){
  int i;
  for (i = 0; i < 3; i++) {
    printf("This is a pthread process.\n");
  }
}

int main(void) {
  int i, ret;
  pthread_t id;

  ret = pthread_create(&id, NULL, (void*) thread, NULL);
  if (ret != 0){
    printf("Create pthread error.\n");
    exit(0);
  }
  for (i = 0; i < 3; i++) {
    printf("This is the main process.\n");
  }
  pthread_join(id, NULL);
  return 0;
}
```
```c
// thread3.c
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>
#include <unistd.h>

void thread (void){
  int i;
  for (i = 0; i < 3; i++) {
    printf("This is a pthread process.\n");
  }
}

int main(void) {
  int i, ret;
  pthread_t id;

  ret = pthread_create(&id, NULL, (void*) thread, NULL);
  if (ret != 0){
    printf("Create pthread error.\n");
    exit(0);
  }
  for (i = 0; i < 3; i++) {
    printf("This is the main process.\n");
  }
  pthread_join(id, NULL);
  return 0;
}
```
## 信号
```c
// signal1.c
#include <signal.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>

int main(int argc, char const *argv[]) {
  pid_t pid;
  int ret;
  pid = fork();
  if (pid < 0){
    perror("Fork error.");
    exit(1);
  }
  else if(pid == 0){
    /*自己给自己发出SIGSTOP信号*/
    raise(SIGSTOP);
    printf("child process exit.\n");
    exit(0);
  }
  else{
    /*在父进程中检测子进程的状态,调用kill函数*/
      printf("pid = %d\n", pid);
      if(waitpid(pid, NULL, WNOHANG) == 0){
        kill(pid, SIGKILL);
        printf("kill %d\n", pid);
      }
  }
  return 0;
}
```
```c
// signal2.c
#include <stdio.h>
#include <unistd.h>

int main(void)
{
  int ret;
  /*调用alarm定时器函数*/
  ret = alarm(5);
  pause();
  printf("I have been waken up.\n ret = %d\n", ret);
  return 0;
}
```
```c
// signal3.c
#include <signal.h>
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>

void my_func(int sig)
{
  if (sig == SIGINT)
    printf("I have got SIGINT\n");
  if (sig == SIGQUIT)
    printf("I have got SIGQUIT\n");
}
int main(void)
{
  printf("Waiting for signal SIGINT or SIGQUIT \n ");
  signal(SIGINT, my_func);
  signal(SIGQUIT, my_func);
  pause();
  exit(0);
}
```
## 文件操作
### 非IO缓存读写
```c
// read.c
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <stdio.h>
// 打开的文件名
#define  SRC_FILE_NAME "/home/rayyu/coursera.txt"
int main(){
  int fd, num;
  char buf[128] = {0};
  // 以只读形式打开文件
  fd = open(SRC_FILE_NAME, O_RDONLY);
  if (fd < 0){
    printf("Open file error\n");
    exit(1);
  }
  // 读取文件的前100个字符并打印
  num = read(fd, buf, 100);
  if (num < 0){
    printf("Read file error\n");
    exit(1);
  }
  printf("%s\n", buf);
  close(fd);
  return 0;
}
```
```c
// write.c
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
// 打开的文件名
#define  DST_FILE_NAME "./demo.txt"
int main(){
  int fd, num;
  char str[] = "hello, world.";
  // 以只读形式打开文件
  fd = open(DST_FILE_NAME, O_WRONLY);
  if (fd < 0){
    printf("Open file error\n");
    exit(1);
  }
  // 将"hello, world"写入文件
  num = write(fd, str, strlen(str));
  if (num < 0){
    printf("write file error\n");
    exit(1);
  }
  printf("write done.\n");
  close(fd);
  return 0;
}
```
```c
// lseek.c
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <stdio.h>
// 打开的文件名
#define  SRC_FILE_NAME "./coursera.txt"
int main(){
  int fd, len;
  // 以只读形式打开文件
  fd = open(SRC_FILE_NAME, O_RDONLY);
  if (fd < 0){
    printf("Open file error\n");
    exit(1);
  }
  // 把文件的位置移到最后, 返回值即为文件大小
  len = lseek(fd, 0, SEEK_END);
  if (len < 0){
    printf("Seek file error\n");
    exit(1);
  }
  printf("file len = %d\n", len);
  close(fd);
  return 0;
}
```
### IO缓存读写
```c
// read1.c
#include <stdlib.h>
#include <stdio.h>
#define  FILE_NAME "./coursera.txt"
int main(){
  FILE *file;
  int num;
  char buf[128] = {0};
  // Read only
  file = fopen(FILE_NAME, "r");
  if (file == NULL){
    printf("Open file error.\n");
    exit(1);
  }
  num = fread(buf, 100, 1, file);
  if (num < 0){
    printf("Fread error.\n");
    exit(1);
  }
  printf("%s\n",buf);
  fclose(file);
  return 0;
}
```
```c
// read2.c
#include <stdlib.h>
#include <stdio.h>
#define  FILE_NAME "./coursera.txt"
int main(){
  FILE *file;
  int num, len;
  char *buf;
  // Read only
  file = fopen(FILE_NAME, "r");
  if (file == NULL){
    printf("open file error.\n");
    exit(1);
  }
  // 将位置指针移动到文件末尾
  fseek(file, 0, SEEK_END);
  // 返回文件位置指针的当前位置
  len = ftell(file);
  if (len < 0){
    printf("ftell error.\n");
    exit(1);
  }
  // 使文件的位置指针返回到文件头
  rewind(file);
  buf = malloc(len);
  num = fread(buf, len, 1, file);
  if (len < 0){
    printf("fread error.\n");
    exit(1);
  }
  printf("%s\n",buf);
  fclose(file);
  return 0;
}
```
```c
// fgets.c
#include <stdlib.h>
#include <stdio.h>
#define  FILE_NAME "./coursera.txt"
int main(){
  FILE *file;
  char buf[256], *plin;
  // Read only
  file = fopen(FILE_NAME, "r");
  if (file == NULL){
    printf("Open file error.\n");
    exit(1);
  }
  while (1) {
    // 每次读取200个字节
    plin = fgets(buf, 200, file);
    if (plin == NULL){
      break;
    }
    printf("%s\n", buf);
  }
  fclose(file);
  return 0;
}
```
```c
// write1.c
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
// 打开的文件名
#define  DST_FILE_NAME "./demo.txt"
int main(){
  int fd, num;
  char str[] = "hello, world.";
  // 以只读形式打开文件
  fd = open(DST_FILE_NAME, O_WRONLY);
  if (fd < 0){
    printf("Open file error\n");
    exit(1);
  }
  // 将"hello, world"写入文件
  num = write(fd, str, strlen(str));
  if (num < 0){
    printf("write file error\n");
    exit(1);
  }
  printf("write done.\n");
  close(fd);
  return 0;
}
```
```c
// write2.c
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#define  FILE_NAME "./demo.txt"
int main(){
  FILE *file;
  int num;
  char str[] = "hello, world.";
  // write and read
  file = fopen(FILE_NAME, "w+");
  if (file == NULL){
    printf("Open file error.\n");
    exit(1);
  }
  num = fwrite(str, strlen(str), 1, file);
  if (num < 0){
    printf("Fread error.\n");
    exit(1);
  }
  fclose(file);
  return 0;
}
```
## 文件锁
```c
// lock.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <errno.h>
#define  REC_FILE_NAME "./test.h"
int main(){
  int file_desc;
  int save_errno;
  file_desc = open(REC_FILE_NAME, O_RDWR | O_CREAT | O_EXCL, 0644);
  if (file_desc == -1){
    save_errno = errno;
    printf("Open failed with error %d.\n", save_errno);
  }
  else{
    printf("Open succeeded.\n" );
  }
  exit(EXIT_SUCCESS);
}
```
### 文件加锁解锁
```c
/* lock_set.c */
 int lock_set(int fd, int type)
 {
     struct flock old_lock, lock;
     lock.l_whence = SEEK_SET;
     lock.l_start = 0;
     lock.l_len = 0;
     lock.l_type = type;
     lock.l_pid = -1;
     /* 判断文件是否可以上锁 */
     fcntl(fd, F_GETLK, &lock);
     if (lock.l_type != F_UNLCK)
     {
         /* 判断文件不能上锁的原因 */
         if (lock.l_type == F_RDLCK) /* 该文件已有读取锁 */
         {
             printf("Read lock already set by %d\n", lock.l_pid);
         }
         else if (lock.l_type == F_WRLCK) /* 该文件已有写入锁 */
         {
             printf("Write lock already set by %d\n", lock.l_pid);
         }
     }
     /* l_type 可能已被F_GETLK修改过 */
     lock.l_type = type;
     /* 根据不同的type值进行阻塞式上锁或解锁 */
     if ((fcntl(fd, F_SETLKW, &lock)) < 0)
     {
         printf("Lock failed:type = %d\n", lock.l_type);
         return 1;
     }
     switch(lock.l_type)
     {
         case F_RDLCK:
         {
             printf("Read lock set by %d\n", getpid());
         }
         break;
         case F_WRLCK:
         {
             printf("Write lock set by %d\n", getpid());
         }
         break;
         case F_UNLCK:
         {
             printf("Release lock by %d\n", getpid());
             return 1;
         }
         break;
         default:
         break;
     }/* end of switch */
     return 0;
 }
```
```c
/* fcntl_read.c */
#include <unistd.h>
#include <sys/file.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <stdio.h>
#include <stdlib.h>
#include "lock_set.c"
int main(void)
{
    int fd;
    fd = open("hello",O_RDWR | O_CREAT, 0644);
    if(fd < 0)
    {
        printf("Open file error\n");
        exit(1);
    }
    /* 给文件上读取锁 */
    lock_set(fd, F_RDLCK);
    getchar();
    /* 给文件解锁 */
    lock_set(fd, F_UNLCK);
    getchar();
    close(fd);
    exit(0);
}
```
```c
/* write_lock.c */
   #include <unistd.h>
   #include <sys/file.h>
   #include <sys/types.h>
   #include <sys/stat.h>
   #include <stdio.h>
   #include <stdlib.h>
   #include "lock_set.c"
   int main(void)
   {
       int fd;
       /* 首先打开文件 */
       fd = open("hello",O_RDWR | O_CREAT, 0644);
       if(fd < 0)
       {
           printf("Open file error\n");
           exit(1);
       }
       /* 给文件上写入锁 */
       lock_set(fd, F_WRLCK);
       getchar();
       /* 给文件解锁 */
       lock_set(fd, F_UNLCK);
       getchar();
       close(fd);
       exit(0);
   }
```
