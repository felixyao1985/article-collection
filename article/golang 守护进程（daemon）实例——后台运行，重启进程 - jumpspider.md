# golang 守护进程（daemon）实例——后台运行，重启进程 - jumpspider
[golang 守护进程（daemon）实例——后台运行，重启进程 - jumpspider - 博客园](https://www.cnblogs.com/jumpspider/p/15912700.html) 

## 需求

1.  **`-daemon`功能**：为任意 Go 程序创建守护进程，使 Go 业务进程脱离终端运行；
2.  **`-forever`功能**：创建监控重启进程，使 Go 业务进程被杀死后能够重启；
3.  不影响业务进程逻辑；
4.  以 Linux 平台为主，其他平台暂不考虑。

## 分析

创建守护进程首先要了解 go 语言如何实现创建进程。在 Unix 中，创建一个进程，通过系统调用 `fork` 实现（及其一些变种，如 `vfork`、`clone`）。在 Go 语言中，Linux 下创建进程使用的系统调用是 `clone` 。

在 C 语言中，通常会用到 2 种创建进程方法：

1.  `fork`

```null
pid = fork();




```

程序会从 `fork` 处一分为二，父进程返回值大于 0，并继续运行；子进程获得父进程的栈、数据段、堆和执行文本段的拷贝，返回值等于 0，并向下继续运行。通过 `fork` 返回值可轻松判断当前处于父进程还是子进程。

但在 Go 语言中，没有直接提供 `fork` 系统调用的封装，如果想只调用 fork，需要通过 `syscall.Syscall(syscall.SYS_FORK, 0, 0, 0)` 实现。

2.  `execve`

```null
execve(pathname, argv, envp);




```

`execve` 为加载一个新程序到当前进程的内存，这将丢弃现存的程序文本段，并为新程序重新创建栈、数据段以及堆。通常将这一动作称为执行一个新程序。

在 Go 语言中，创建进程方法主要有 3 种：

1.  `exec.Command`

```null
package main

import (
    "os"
    "os/exec"
    "path/filepath"
    "time"
)

func main() {
    
    if os.Getppid() != 1 {
        
        filePath, _ := filepath.Abs(os.Args[0])
        cmd := exec.Command(filePath, os.Args[1:]...)
        
        cmd.Stdin = os.Stdin 
        cmd.Stdout = os.Stdout
        cmd.Stderr = os.Stderr
        cmd.Start() 
        os.Exit(0)
        
    } else {
        
        time.Sleep(time.Second * 10)
    }
}

```

2.  `os.StartProcess`

```null
if os.Getppid()!=1{   
	args:=append([]string{filePath},os.Args[1:]...)
	os.StartProcess(filePath,args,&os.ProcAttr{Files:[]*os.File{os.Stdin,os.Stdout,os.Stderr}})
	os.Exit(0)
}

```

3.  `syscall.RawSyscall(syscall.SYS_FORK, 0, 0, 0)`

```null
pid, _, sysErr := syscall.RawSyscall(syscall.SYS_FORK, 0, 0, 0)
if sysErr != 0 {
	Utils.LogErr(sysErr)
	os.Exit(0)
}

```

可以参考例子：[https://studygolang.com/articles/3597](https://studygolang.com/articles/3597)

在分析这个例子的时候，方法 1 和 2 针对于如何判断该进程是否是子进程，例子的方法是 `os.Getppid()!=1`，也就是默认了父进程退出之后，子进程会被 1 号进程接管。

但在我 `Ubuntu Desktop` 本地测试时却发现，接管孤儿进程的并不是 1 号进程，因此考虑到程序稳定性和兼容性，不能够以 `ppid` 作为判断父子进程的依据。

方法 3 直接进行了系统调用，虽然可以通过 `pid` 进行判断父子进程，但该方法过于底层，例子中也没有推荐使用，所以也没有采纳。

### 1. 守护进程

考虑利用方法 1 进行进程创建，由于 `exec.Command` 包含了参数传递，可以通过传入不同的参数，实现判断启动守护进程还是启动业务进程。

```null
func main(){
	daemon := flag.Bool("daemon", false, "run in daemon")
	if *daemon {
		cmd := exec.Command(os.Args[0])
		cmd.Stdin = os.Stdin
		cmd.Stdout = os.Stdout
		cmd.Stderr = os.Stderr
		err := cmd.Start()
		if err != nil {
			fmt.Fprintf(os.Stderr, "[-] Error: %s\n", err)
		}
		os.Exit()
	} else {
		DoSomething()
	}
}

```

运行时存在 `-daemon` 参数，则运行 `exec.Command` 生成子进程，在传参时删掉 `-daemon` 参数，再次进入 `main` 时就进入子进程业务逻辑了，这时父进程也退出，子进程就被系统进程接管。

### 2. 重启进程

通过上述分析，基本已经实现了守护进程创建，重启进程就依葫芦画瓢了。

```null
forever := flag.Bool("forever", false, "run forever")
if *forever{
	for {
		cmd := exec.Command(args[0])
		cmd.Stdin = os.Stdin
		cmd.Stdout = os.Stdout
		cmd.Stderr = os.Stderr
		err := cmd.Start()
		if err != nil {
			fmt.Fprintf(os.Stderr, "[-] Error: %s\n", err)
		}
		cmd.Wait()
	}
}

```

在死循环（`for{...}`）中，创建新的业务进程，通过 `cmd.Wait()` 等待业务进程退出状态，如果业务进程退出，则再次循环创建进程。

By the way，重启进程和守护进程是可以解耦合的，可以单独判断参数 `-daemon` 和 `-forever`，不必再进行参数个数判断。

## 实现

本次实现主要通过方法 1 进行进程创建，以 main 函数作为程序入口点，通过传参数不同，来判断父子进程，这样有 2 个好处：

1.  参数不同实现启动不同进程；
2.  守护进程和重启进程对业务进程透明，不影响业务进程逻辑。

直接上代码

### go-daemon.go

```null
package main

import (
	"os"
	"os/exec"
	"fmt"
	"flag"
	"log"
	"time"
)

const (
	DAEMON = "daemon"
	FOREVER = "forever"
)

func DoSomething(){
	fp, _ := os.OpenFile("./dosomething.log", os.O_WRONLY|os.O_CREATE|os.O_APPEND, 0644)
	log.SetOutput(fp)
	for{
		log.Printf("DoSomething running in PID: %d PPID: %d\n", os.Getpid(), os.Getppid())
		time.Sleep(time.Second * 5)
	}
}

func StripSlice(slice []string, element string) []string {
	for i := 0; i < len(slice); {
		if slice[i] == element && i != len(slice)-1 {
			slice = append(slice[:i], slice[i+1:]...)
		} else if slice[i] == element && i == len(slice)-1 {
			slice = slice[:i]
		} else {
			i++
		}
	}
	return slice
}

func SubProcess(args []string) *exec.Cmd {
	cmd := exec.Command(args[0], args[1:]...)
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr
	err := cmd.Start()
	if err != nil {
		fmt.Fprintf(os.Stderr, "[-] Error: %s\n", err)
	}
	return cmd
}

func main(){
	daemon := flag.Bool(DAEMON, false, "run in daemon")
	forever := flag.Bool(FOREVER, false, "run forever")
	flag.Parse()
	fmt.Printf("[*] PID: %d PPID: %d ARG: %s\n", os.Getpid(), os.Getppid(), os.Args)
	if *daemon {
		SubProcess(StripSlice(os.Args, "-"+DAEMON))
		fmt.Printf("[*] Daemon running in PID: %d PPID: %d\n", os.Getpid(), os.Getppid())
		os.Exit(0)
	} else if *forever {
		for {
			cmd := SubProcess(StripSlice(os.Args, "-"+FOREVER))
			fmt.Printf("[*] Forever running in PID: %d PPID: %d\n", os.Getpid(), os.Getppid())
			cmd.Wait()
		}
		os.Exit(0)
	} else {
		fmt.Printf("[*] Service running in PID: %d PPID: %d\n", os.Getpid(), os.Getppid())
	}
	DoSomething()
}


```

## 使用

### 编译

```null
go build -ldflags "-s -w" go-daemon.go

```

### 运行

```null
./go-daemon -daemon -forever

```

## 代码及参考

-   [http://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter10/10.1.html](http://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter10/10.1.html)
-   [https://studygolang.com/articles/3597](https://studygolang.com/articles/3597)
-   [https://blog.csdn.net/weixin_43204126/article/details/96871493](https://blog.csdn.net/weixin_43204126/article/details/96871493)
-   [http://cn.voidcc.com/question/p-hbkkfkzd-qw.html](http://cn.voidcc.com/question/p-hbkkfkzd-qw.html)
