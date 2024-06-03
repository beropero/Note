## 流式接收

```js
 const url = 'http://localhost:8080/ai/sendmsg';
        const params = new URLSearchParams();
        params.append('message', '写一个快速排序');
        params.append('key', 'beropero');
        // 将参数附加到 URL
        const requestUrl = `${url}?${params.toString()}`;

        // 发起 GET 请求
        fetch(requestUrl)
        .then(response => {
            // 检查响应是否成功
            if (!response.ok) {
                throw new Error('Failed to fetch data:', response.statusText);
            }

            // 创建一个 ReadableStream
            const reader = response.body.getReader();
            const decoder = new TextDecoder();

            // 逐步处理流数据
            function readStream() {
                reader.read().then(({ done, value }) => {
                    if (done) {
                        console.log('Stream ended');
                        return;
                    }
                    // 将二进制数据转换为文本
                    const text = decoder.decode(value);
                    console.log('Received chunk:', text);
                    // 继续读取流数据
                    readStream();
                });
            }

            // 开始读取流数据
            readStream();
        })
        .catch(error => {
            console.error(error);
        });
```

