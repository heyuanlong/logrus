源码分析：

1. 使用 *sync.Pool 来缓存 bytes.Buffer
```Go
	buffer = bufPool.Get()
	defer func() {
		newEntry.Buffer = nil
		buffer.Reset()
		bufPool.Put(buffer)
	}()
	buffer.Reset()
	newEntry.Buffer = buffer

	newEntry.write()

	newEntry.Buffer = nil
```

2. entry是每次写日志都是一个新的实例
3. Logger 是基本日志配置对象
4. exported.go 暴露一个默认日志配置对象 std = New()
5. logrus.go 就定义了Level和接口

使用：
```Go
    l := logrus.New()
	l.SetReportCaller(true)
	l.Out = os.Stdout
	l.Formatter = &logrus.TextFormatter{
		TimestampFormat : "2006-01-02 15:04:05",
		DisableTimestamp: false,
		CallerPrettyfier: func(f *runtime.Frame) (string, string) {
			//s := strings.Split(f.Function, ".")
			//funcname := s[len(s)-1]
			//_, filename := path.Split(f.File)
			i:=strings.LastIndex(f.File, "/")
			j:=strings.LastIndex(f.File[0:i], "/")
			return "", fmt.Sprintf("%s:%d", f.File[j:], f.Line)
		},
		SortingFunc: func(keys []string) {
			sort.Slice(keys, func(i, j int) bool {
				if keys[j] == logrus.FieldKeyTime {
					return false
				}
				if keys[i] == logrus.FieldKeyTime {
					return true
				}

				if keys[j] == logrus.FieldKeyFile {
					return false
				}
				if keys[i] == logrus.FieldKeyFile {
					return true
				}

				return strings.Compare(keys[i], keys[j]) == -1
			})
		},
	}
	l.Info("example of custom format caller")
```