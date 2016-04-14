## How to use customized `gin` in codoon?

### Features
* Graceful exit
* Change log level at runtime
* Support custom handlers

### Sample Code


```go
package main

import (
	"net/http"
	"os"
	"third/gin"

	"third/go-logging"
)

var log *logging.Logger = logging.MustGetLogger("testlog")

func initLog() []gin.LoggerInfo {
	backend := logging.NewLogBackend(os.Stdout, "", 0)
	leveledBackend := logging.AddModuleLevel(backend)
	leveledBackend.SetLevel(logging.INFO, "")
	return []gin.LoggerInfo{
		gin.LoggerInfo{
			Name:    "defautllogger",
			LLogger: leveledBackend,
		},
	}
}

func hiHandler(c *gin.Context) {
	c.JSON(http.StatusOK, gin.H{"rsp": "hi"})
}

func main() {
	loggers := initLog()

	handlers := []gin.HandlerInfo{
		gin.HandlerInfo{
			Method:  "GET",
			Path:    "/hi",
			Handler: hiHandler,
		},
	}
	ginengine := gin.UseAdminServer(":8082", loggers, handlers)
	// By default, HandleSignal captures interrupt, kill signals.
	// Your can pass other signas to it.
	ginengine.HandleSignal()

}
```

### Usage

* Graceful exit: send intterupt(`Ctrl+C`)/kill(`kill -9 pid`) signal to process or `curl http://localhost:8082/admin/gracefulexit` 
* Show log level: `http://localhost:8082/admin/show_log_level`

```json
{
  "Data": [
    {
      "levels": [
        {
          "level": "INFO",
          "module": ""
        }
      ],
      "name": "defautllogger"
    }
  ],
  "Description": "",
  "Status": "OK"
}
```

* Change log level: `curl http://localhost:8082/admin/set_log_level -d 'name=defautllogger&level=DEBUG'`

```json
{
  "Data": [
    {
      "levels": [
        {
          "level": "DEBUG",
          "module": ""
        }
      ],
      "name": "defautllogger"
    }
  ],
  "Description": "",
  "Status": "OK"
}
```

* Call custom handler: `curl http://localhost:8082/hi`

### Limitations
* `multiLogger` of `go-logging` do NOT support change log level at runtime