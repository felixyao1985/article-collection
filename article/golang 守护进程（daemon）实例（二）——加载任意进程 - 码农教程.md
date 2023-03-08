# golang 守护进程（daemon）实例（二）——加载任意进程 - 码农教程
[golang 守护进程（daemon）实例（二）——加载任意进程 - 码农教程](http://www.manongjc.com/detail/28-moxxrzaowfyucrz.html) 

## 前期实现

1.  **`-daemon`功能**：为任意 Go 程序创建守护进程，使 Go 业务进程脱离终端运行；
2.  **`-forever`功能**：创建监控重启进程，使 Go 业务进程被杀死后能够重启；
3.  不影响业务进程逻辑；
4.  实现 Linux 端运行。

见上一篇文章[golang 守护进程（daemon）实例——后台运行，重启进程](https://www.cnblogs.com/jumpspider/p/15912700.html)

## 更进一步思考

**我们可以不局限于 Go 代码层面的守护进程，而是实现对任意进程的守护进程创建，从而对所有 Linux 环境的程序都可以进行加载。** 

## 需求

1.  **`-prog`功能**：将任意待执行的程序以参数形式传入；
2.  不影响业务进程逻辑，也就是把原先的 `DoSomething` 部分修改为启动新进程；
3.  跨平台，实现 Linux 端 / Windows 端执行。

## 实现

### 关键点

1.  为实现兼容不同操作系统，对操作系统类型进行了判断

**注意：在 Windows 环境下，虽然 `daemon` 和 `forever` 两项功能都可以使用，但 Windows 没有所谓的 daemon 托管机制，所以 `daemon` 功能只是把业务进程变成了后台运行，`cmd` 挂掉程序依旧会退出。** 

```null
switch runtime.GOOS { 
	case "darwin": 
		cmd = exec.Command(os.Getenv("SHELL"), "-c", args[0])
		break
	case "linux": 
		cmd = exec.Command(os.Getenv("SHELL"), "-c", args[0])
		break
	case "windows": 
		cmd = exec.Command("cmd", "/C", args[0])
		break
	default:
		os.Exit(0)
		break
}

```

2.  在启动业务进程时，要记得进行 `cmd.Wait()`，否则业务进程尚未结束父进程就挂掉了，业务进程就无条件变成后台进程了

```null
cmd = SubProcess([]string{*program}, true)
cmd.Wait()

```

3.  这个点存在一个 Bug，但不影响程序使用。在完成 `forever` 功能时，当业务进程第一次被杀死后，`for`循环中重启业务进程，但 `-prog` 参数被传递了 2 次，百思不得其解，希望有大神能帮忙解决一下

```null
for {
	fmt.Printf("[*] Forever running in PID: %d PPID: %d\n", os.Getpid(), os.Getppid())
	cmd = SubProcess(StripSlice(args, "-"+FOREVER), false)
	cmd.Wait()
}

```

### go-daemon-loader.go

直接上代码

```null
package main

import (
	"os"
	"os/exec"
	"fmt"
	"flag"
	"runtime"
)

const (
	PROGRAM = "prog"
	DAEMON = "daemon"
	FOREVER = "forever"
)

func StripSlice(slice []string, element string) []string {
	for i := 0; i < len(slice); {
		if slice[i] == element && i != len(slice)-1 {
			slice = append(slice[:i], slice[i+1:]...)
			break
		} else if slice[i] == element && i == len(slice)-1 {
			slice = slice[:i]
			break
		} else {
			i++
		}
	}
	return slice
}

func SubProcess(args []string, shell bool) *exec.Cmd {
	var cmd *exec.Cmd
	if shell { 
		switch runtime.GOOS { 
		case "darwin": 
			cmd = exec.Command(os.Getenv("SHELL"), "-c", args[0])
			break
		case "linux": 
			cmd = exec.Command(os.Getenv("SHELL"), "-c", args[0])
			break
		case "windows": 
			cmd = exec.Command("cmd", "/C", args[0])
			break
		default:
			os.Exit(1)
			break
		}
	} else {
		cmd = exec.Command(args[0], args[1:]...)
	}
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
	var cmd *exec.Cmd
	program := flag.String(PROGRAM, "", "run program")
	daemon := flag.Bool(DAEMON, false, "run in daemon")
	forever := flag.Bool(FOREVER, false, "run forever")
	flag.Parse()
	fmt.Printf("[*] PID: %d PPID: %d ARG: %s PROG:\"%s\"\n", os.Getpid(), os.Getppid(), os.Args, *program)
	if *program == "" {
		flag.Usage()
		os.Exit(1)
	}
	if *daemon {
		fmt.Printf("[*] Daemon running in PID: %d PPID: %d\n", os.Getpid(), os.Getppid())
		SubProcess(StripSlice(os.Args, "-"+DAEMON), false)
		os.Exit(0)
	} else if *forever {
		args := os.Args
		for {
			fmt.Printf("[*] Forever running in PID: %d PPID: %d\n", os.Getpid(), os.Getppid())
			cmd = SubProcess(StripSlice(args, "-"+FOREVER), false)
			cmd.Wait()
		}
		os.Exit(0)
	} else {
		fmt.Printf("[*] Service running in PID: %d PPID: %d\n", os.Getpid(), os.Getppid())
		cmd = SubProcess([]string{*program}, true)
		cmd.Wait()
	}
}


```

## 使用

### 编译

-   Linux 编译（同 MacOS 编译，未测试）

```null
go build -ldflags "-s -w" ./go-daemon-loader.go

```

-   Windows 编译

```null
go build -ldflags "-s -w" .\go-daemon-loader.go

```

### 运行

-   Linux

```null
./go-daemon-loader -prog "ping 127.0.0.1" -daemon -forever

```

-   Windows

```null
.\go-daemon-loader.exe -prog "ping 127.0.0.1 -t" -daemon -forever

```
