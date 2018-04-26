## client

1.多goroutine共享，这意味着在别处对DefaultClient的改动会影响你当前的使用

2.未设置connection timeout和 read/write timeout

3.默认的idle connection等设置可能不满足你的需求

4.如果再loop中使用可能会导致内存泄漏

```
var timeout = time.Duration(2 * time.Second)
func dialTimeout(network, addr string) (net.Conn, error) {
    return net.DialTimeout(network, addr, timeout)
}
func main() {
    transport := http.Transport{
        Dial: dialTimeout,
        Proxy: ...,
        MaxIdleConns: ...,
        MaxIdleConnsPerHost: ...,
        IdleConnTimeout: ...,
        ResponseHeaderTimeout: ...,
        DisableCompression:...,
    }
    client := http.Client{
        Transport: &transport,
    }
    resp, err := client.Get("http://some.url")
}
```