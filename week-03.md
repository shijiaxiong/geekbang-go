## Q1: 基于 errgroup 实现一个 http server 的启动和关闭 ，以及 linux signal 信号的注册和处理，要保证能够一个退出，全部注销退出。

```go
package main

import (
	"context"
	"fmt"
	"io"
	"net/http"
	"os"
	"os/signal"

	"golang.org/x/sync/errgroup"
)

func main() {
	ctx, cancel := context.WithCancel(context.Background())

	group, errCtx := errgroup.WithContext(ctx)

	// 启动服务
	server := &http.Server{Addr: ":80"}

	group.Go(func() error {
		return StartHttpServer(server)
	})

	group.Go(func() error {
		// 阻塞等待
		<-errCtx.Done()

		// 关闭http server
		return server.Shutdown(errCtx)
	})

	// 监听信号
	channel := make(chan os.Signal, 1)
	signal.Notify(channel)

	group.Go(func() error {
		for {
			select {
			case <-errCtx.Done():
				return errCtx.Err()
			case <-channel: // 信号终止
				cancel()
			}
		}
	})

	if err := group.Wait(); err != nil {
		fmt.Println("group error", err.Error())
	}

	fmt.Println("mission success！")
}

func StartHttpServer(server *http.Server) error {
	http.HandleFunc("/hello", func(w http.ResponseWriter, request *http.Request) {
		_, _ = io.WriteString(w, "hello world!")
	})

	return server.ListenAndServe()
}

```

