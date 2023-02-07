### grpc 指南

##### grpc 架构图

![grpc 架构](https://grpc.io/img/grpc_concept_diagram_00.png "grpc")

参考文档：
[grpc 指南](https://doc.oschina.net/grpc?t=58008)

##### grpc 使用

1. 定义服务接口及参数（proto 文件)

```
syntax = "proto3";

option java_package = "io.grpc.examples";

package helloworld;

// The greeter service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

2. 生成 gRPC 代码

一旦定义好服务，我们可以使用 protocol buffer 编译器 protoc 来生成创建应用所需的特定客户端和服务端的代码 - 你可以生成任意 gRPC 支持的语言的代码。生成的代码同时包括客户端的存根和服务端要实现的抽象接口，均包含 Greeter 所定义的方法。

以下为 生成的Java代码 
```
@javax.annotation.Generated(
    value = "by gRPC proto compiler (version 1.31.1)",
    comments = "Source: service_demo.proto")
public final class GreetingServiceGrpc {

  private GreetingServiceGrpc() {}

  public static final String SERVICE_NAME = "org.example.proto.GreetingService";


  /**
   * <pre>
   * 4. service, unary request/response
   * </pre>
   */
  public static abstract class GreetingServiceImplBase implements io.grpc.BindableService {

    /**
     */
    public void greeting(org.example.proto.Grpc.HelloRequest request,
        io.grpc.stub.StreamObserver<org.example.proto.Grpc.HelloResponse> responseObserver) {
      asyncUnimplementedUnaryCall(getGreetingMethod(), responseObserver);
    }

    @java.lang.Override public final io.grpc.ServerServiceDefinition bindService() {
      return io.grpc.ServerServiceDefinition.builder(getServiceDescriptor())
          .addMethod(
            getGreetingMethod(),
            asyncUnaryCall(
              new MethodHandlers<
                org.example.proto.Grpc.HelloRequest,
                org.example.proto.Grpc.HelloResponse>(
                  this, METHODID_GREETING)))
          .build();
    }
  }

  /**
   * <pre>
   * 4. service, unary request/response
   * </pre>
   */
  public static final class GreetingServiceStub extends io.grpc.stub.AbstractAsyncStub<GreetingServiceStub> {
    private GreetingServiceStub(
        io.grpc.Channel channel, io.grpc.CallOptions callOptions) {
      super(channel, callOptions);
    }

    @java.lang.Override
    protected GreetingServiceStub build(
        io.grpc.Channel channel, io.grpc.CallOptions callOptions) {
      return new GreetingServiceStub(channel, callOptions);
    }

    /**
     */
    public void greeting(org.example.proto.Grpc.HelloRequest request,
        io.grpc.stub.StreamObserver<org.example.proto.Grpc.HelloResponse> responseObserver) {
      asyncUnaryCall(
          getChannel().newCall(getGreetingMethod(), getCallOptions()), request, responseObserver);
    }
  }

  /**
   * <pre>
   * 4. service, unary request/response
   * </pre>
   */
  public static final class GreetingServiceBlockingStub extends io.grpc.stub.AbstractBlockingStub<GreetingServiceBlockingStub> {
    private GreetingServiceBlockingStub(
        io.grpc.Channel channel, io.grpc.CallOptions callOptions) {
      super(channel, callOptions);
    }

    @java.lang.Override
    protected GreetingServiceBlockingStub build(
        io.grpc.Channel channel, io.grpc.CallOptions callOptions) {
      return new GreetingServiceBlockingStub(channel, callOptions);
    }

    /**
     */
    public org.example.proto.Grpc.HelloResponse greeting(org.example.proto.Grpc.HelloRequest request) {
      return blockingUnaryCall(
          getChannel(), getGreetingMethod(), getCallOptions(), request);
    }
  }

  /**
   * <pre>
   * 4. service, unary request/response
   * </pre>
   */
  public static final class GreetingServiceFutureStub extends io.grpc.stub.AbstractFutureStub<GreetingServiceFutureStub> {
    private GreetingServiceFutureStub(
        io.grpc.Channel channel, io.grpc.CallOptions callOptions) {
      super(channel, callOptions);
    }

    @java.lang.Override
    protected GreetingServiceFutureStub build(
        io.grpc.Channel channel, io.grpc.CallOptions callOptions) {
      return new GreetingServiceFutureStub(channel, callOptions);
    }

    /**
     */
    public com.google.common.util.concurrent.ListenableFuture<org.example.proto.Grpc.HelloResponse> greeting(
        org.example.proto.Grpc.HelloRequest request) {
      return futureUnaryCall(
          getChannel().newCall(getGreetingMethod(), getCallOptions()), request);
    }
  }
}

```

3. 编写服务端对应的业务逻辑

服务端代码一般都是继承自 grpc生成的GreetingServiceGrpc.GreetingServiceImplBase

```
    public static class GreetingServiceImpl extends GreetingServiceGrpc.GreetingServiceImplBase {
        @Override
        public void greeting(Grpc.HelloRequest request, StreamObserver<Grpc.HelloResponse> responseObserver) {
            System.out.println(request);

            String greeting = "Hello there, " + request.getName();

            Grpc.HelloResponse response = Grpc.HelloResponse.newBuilder().setGreeting(greeting).build();

            responseObserver.onNext(response);
            responseObserver.onCompleted();
        }
    }

```

4. 服务端(基于http2 协议)

使用 grpc-api 包中的ServerBuilder，构建服务端
```
        Server server = ServerBuilder.forPort(9988)
                .addService(new GreetingServiceImpl()).build();

        System.out.println("Starting server...");
        server.start();
        System.out.println("Server started!");
        server.awaitTermination();

```

5. 客户端(基于http2 协议)

使用 grpc-api 包中的 ManagedChannelBuilder 构建channel通道，然后使用grpc 生成的stub 代码，进行远程调用
```
ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 9988)
                .usePlaintext()
                .build();

        GreetingServiceGrpc.GreetingServiceBlockingStub stub =
                GreetingServiceGrpc.newBlockingStub(channel);

        Grpc.HelloResponse helloResponse = stub.greeting(
                Grpc.HelloRequest.newBuilder()
                        .setName("Ray")
                        .setAge(18)
                        .setSentiment(Grpc.Sentiment.HAPPY)
                        .build());

```