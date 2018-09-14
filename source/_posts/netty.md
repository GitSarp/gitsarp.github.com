---
title: netty——持续更新
date: 2018-08-29 23:15:48
tags: netty
categories: Java
---
#### netty——持续更新
- [maven依赖](https://mvnrepository.com/artifact/io.netty)
- what is netty?

The Netty project is an effort to provide an asynchronous event-driven network application framework and tooling for the rapid development of maintainable high-performance · high-scalability protocol servers and clients.

#### Get Start
- ServerHandler

```
public class DiscardServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
//        ((ByteBuf)msg).release();//discard everything

        //官方说明，msg必须要释放！！！
//        ByteBuf in = (ByteBuf) msg;
//        try {
//            //System.out.println(in.toString(CharsetUtil.US_ASCII));
//            while (in.isReadable()) {
//                System.out.println((char) in.readByte());
//                System.out.flush();
//            }
//        } finally {
//            ReferenceCountUtil.release(msg);
//        }

        ctx.write(msg);//使用ctx则不需要对msg进行释放，netty会帮我们释放
        ctx.flush();
        //ctx.writeAndFlush(msg);//more brevity
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        //the caught exception should be logged and its associated channel should be closed here
        //you might want to send a response message with an error code before closing the connection.
        cause.printStackTrace();
        ctx.close();
    }
}
```
- Server

```
public class DiscardServer {
    private int port;

    public DiscardServer(int port) {
        this.port = port;
    }

    public void run()throws Exception {

        NioEventLoopGroup boss = new NioEventLoopGroup();//parent group,accept con
        NioEventLoopGroup worker = new NioEventLoopGroup();//child group

        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap();//server
            serverBootstrap.group(boss, worker)
                    .channel(NioServerSocketChannel.class)//channel
                    .childHandler(new ChannelInitializer<SocketChannel>() {//init channel
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            socketChannel.pipeline().addLast(new DiscardServerHandler());//add handler
                        }
                    })
                    .option(ChannelOption.SO_BACKLOG, 128)
                    .childOption(ChannelOption.SO_KEEPALIVE, true);

            ChannelFuture channelFuture = serverBootstrap.bind(port).sync();
            channelFuture.channel().closeFuture().sync();
        } finally {
            worker.shutdownGracefully();
            boss.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws Exception{
        int port = 0;
        if(args.length>0) {
            Integer.parseInt(args[0]);
        }else {
            port = 8080;
        }
        new DiscardServer(port).run();
    }
}
```

- [NioEventLoopGroup](http://netty.io/4.1/api/io/netty/channel/nio/NioEventLoopGroup.html)

The first one, often called 'boss', accepts an incoming connection. The second one, often called 'worker', handles the traffic of the accepted connection once the boss accepts the connection and registers the accepted connection to the worker. How many Threads are used and how they are mapped to the created Channels depends on the **EventLoopGroup** implementation and may be even configurable via a constructor.

- [ServerBootstrap](http://netty.io/4.1/api/io/netty/bootstrap/ServerBootstrap.html)

ServerBootstrap is a helper class that sets up a server. You can set up the server using a Channel directly. However, please note that this is a tedious process, and you do not need to do that in most cases.

- [ChannelInitializer](http://netty.io/4.1/api/io/netty/channel/ChannelInitializer.html)

The ChannelInitializer is a special handler that is purposed to help a user configure a new Channel. It is most likely that you want to configure the ChannelPipeline of the new Channel by adding some handlers such as DiscardServerHandler to implement your network application. As the application gets complicated, it is likely that you will add more handlers to the pipeline and extract this anonymous class into a top level class eventually.

Please refer to the apidocs of ChannelOption and the specific ChannelConfig implementations to get an overview about the supported ChannelOptions.

option() is for the NioServerSocketChannel that accepts incoming connections. childOption() is for the Channels accepted by the parent ServerChannel, 