---
title: Java socket编程
date: 2018-07-11 22:13:38
tags: JavaSocket
categories: Java
---
### 传统阻塞方式
直接上代码
- 客户端
```
public class MyClient {
    String host;
    int port;
    Socket socket;
    int timeOut=10000;
    BufferedReader bufferedReader;
    PrintStream printStream;
    BufferedReader receive;

    public MyClient(String host, int port) {
        this.host = host;
        this.port = port;
        try {
            socket=new Socket(host,port);
            socket.setSoTimeout(timeOut);//设置接收响应的超时时间
            bufferedReader=new BufferedReader(new InputStreamReader(System.in));//用户输入
            printStream=new PrintStream(socket.getOutputStream());//响应流
            receive=new BufferedReader(new InputStreamReader(socket.getInputStream()));//响应流
        }catch (Exception e){

        }
    }
<!--more-->
    public void sendAndReceive() throws Exception{
        boolean bye=false;
        String message;
        while (!bye){
            System.out.println("input your message...");
            message=bufferedReader.readLine();
            if(message.equals("")||message.isEmpty())
                break;
            if(message.equals("bye"))
                bye=true;
            printStream.println(message);
            try{
                message=receive.readLine();
                System.out.println("receive:"+message);
            }catch (Exception e){
                System.out.println("response timeout");
                break;
            }
        }
        printStream.close();
        try {
            socket.close();
        }catch (Exception e){}
    }

    public static void main(String[] args){
        MyClient myClient=new MyClient("127.0.0.1",8085);
        //MyClient myClient2=new MyClient("127.0.0.1",8085);
        try {
            myClient.sendAndReceive();
            //myClient2.sendAndReceive();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
- 服务端线程
```
public class MyServerThread implements Runnable{
    Socket client;
    BufferedReader req;
    String reqMsg;
    PrintStream out;

    public MyServerThread(Socket client) {
        this.client = client;
    }

    @Override
    public void run() {
        boolean endFlag=true;
        while (endFlag){
            try {
                req=new BufferedReader(new InputStreamReader(client.getInputStream()));
                reqMsg=req.readLine();
                if(reqMsg==null||reqMsg.equals("bye"))
                    endFlag=false;
                out=new PrintStream(client.getOutputStream());
                out.println("server resp:"+reqMsg);
            } catch (IOException e) {
                e.printStackTrace();
                endFlag=false;
            }
        }
        out.close();
        try {
            client.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
- 服务端
```
public class MyServer {
    public static void main(String[] args) {
        Socket client;
        try {
            ServerSocket serversocket = new ServerSocket(8085);
            while (true){
                client=serversocket.accept();
                new Thread(new MyServerThread(client)).start();
            }
        }catch (Exception e){

        }
    }
}
```