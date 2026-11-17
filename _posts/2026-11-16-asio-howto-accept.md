---
layout: post
title:  asio如何正确accept
date:   2026-11-16 15:27:17
categories: Network
tags: asio
---

* content
{:toc}


# 正确使用acceptor

asio acceptor有通过endpoint构造的重载函数，创建acceptor之后就可以accept了，代码如下：

```
using asio::ip::tcp;
tcp::acceptor acceptor(io_context, tcp::endpoint(tcp::v4(), port));
acceptor.async_accept(socket, [](asio::error_code ec){});
```

这个代码在一般情况下是没问题的，但是如果端口被占用的时候就会抛异常，所以用这种方法去accept的时候需要注意捕获异常。

如果不希望抛异常的话，则需要先先监听端口再accept，如果监听失败就返回错误码。

```
tcp::acceptor acceptor(io_context);
acceptor.open(endpoint.protocol());
acceptor.set_option(boost::asio::ip::tcp::acceptor::reuse_address(true));
asio::error_code ec;
acceptor.bind(endpoint, ec);
if(ec) {
    std::cout<<"bind port failed "<<ec.message()<<"\n";
    return ec;
}
acceptor.listen();
```

这里面需要设置一下reuse\_address option，为什么要设置它？这个选项对于重启服务器的时候有帮助，当socket处于TIME\_WAIT状态时，启动服务器，就会bind不成功，报：Address already in use，如果设置了reuse\_address(true)，则可以bind成功。为什么第一个例子中没有显式去设置这个参数呢，因为通过endpoint构造的acceptor默认参数就是`reuse_addr = true`。

# 正确关闭acceptor

我们写异步server的时候，需要通过异步循环调用async\_accept:

```
void server::do_accept()
{
  acceptor_.async_accept(
      [this](std::error_code ec, asio::ip::tcp::socket socket)
      {
        if (!ec)
        {
          connection_manager_.start(std::make_shared<connection>(
              std::move(socket), connection_manager_, request_handler_));
        }else {
            std::cout<<ec.message()<<"\n";
        }

        do_accept();
      });
}
```

这里需要注意的是错误处理，这里不像异步读写那样，在收到错误的时候并没有return，而是继续accept，因为server在某一次accept失败的时候它还可以继续accept其它连接，如果return了，则整个server不能响应新的连接了。但是上面的这个代码的错误处理还不够，当acceptor被关闭之后整个accept就会陷入一个死循环，一直会accept错误，所以正确错误处理方法是accept之前判断一下acceptor是否已经关闭了。

```
void server::do_accept()
{
  acceptor_.async_accept(
      [this](std::error_code ec, asio::ip::tcp::socket socket)
      {

        if (!acceptor_.is_open())
        {
          return;
        }

        if (!ec)
        {
          connection_manager_.start(std::make_shared<connection>(
              std::move(socket), connection_manager_, request_handler_));
        }else {
            std::cout<<ec.message()<<"\n";
        }

        do_accept();
      });
}
```

再来说关闭acceptr，很多人可能会写这样的代码来关闭acceptor：

```
void close() {
    std::error_code ec
    acceptor_.close(ec);
}
```

这样直接关闭acceptor的做法是错误的，会引起不可预测的后果，因为acceptor正在异步accept，这时在另外的线程中去关闭它，多线程操作acceptor会导致ub行为，表现为有时候async\_accept会返回错误，有时候则会hang主，不会返回错误。在异步回调模式下这个错误造成的影响似乎不太大，如果在协程环境下，会导致异步accept的协程永远不会返回，这时候如果再重启服务端则会出现客户端连接成功，但read永远超时的情况，这个问题的排查花了我半天时间。

正确的关闭acceptor的方法是避免多线程去操作acceptor，通过post去关闭它。

```
void close() {
    asio::post(acceptor_.get_executor(), []{
        std::error_code ec
        acceptor_.close(ec);
    });
}
```

post到acceptor的io\_context里则可以保证在同一个线程里操作acceptor，确保正确关闭它。
