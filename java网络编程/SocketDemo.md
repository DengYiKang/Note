# SocketDemo

## 简单的一对一

### 服务器端

```java
public class TcpServer {
    public static void main(String[] args) {
        try {
            ServerSocket serverSocket = new ServerSocket(65000);
            Socket socket = serverSocket.accept();
            new TcpServerThread(socket).run();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    static class TcpServerThread extends Thread {
        private Socket socket;

        public TcpServerThread(Socket socket) {
            this.socket = socket;
        }

        @Override
        public void run() {
            try {
                System.out.println("server start...");
                InputStream inputStream = socket.getInputStream();
                InputStreamReader inputStreamReade = new InputStreamReader(inputStream);
                BufferedReader bufferedReader = new BufferedReader(inputStreamReade);
                OutputStream outputStream = socket.getOutputStream();
                PrintWriter printWriter = new PrintWriter(outputStream);
                String result = "";
                //bufferedReader.readLine()只有到达流末尾在会返回null，客户端未发数据时是阻塞的
                while ((result = bufferedReader.readLine()) != null) {
                    System.out.println(result);
                    //printWriter.println(result.length());
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### 客户端

```java
public class TcpClient {
    public static void main(String[] args) {
        try {
            Socket socket=new Socket("127.0.0.1", 65000);
            InputStream inputStream=socket.getInputStream();
            OutputStream outputStream=socket.getOutputStream();
            //注意PrintWriter构造函数的第二个参数要设置为true，自动flush
            PrintWriter printWriter=new PrintWriter(outputStream, true);
            Scanner sc=new Scanner(System.in);
            String msg="";
            while(true){
                msg=sc.nextLine();
                printWriter.println(msg);
            }
        } catch (UnknownHostException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```