nginx 负载均衡

疑问：
1. 因为访问的都是同一个IP，所以，我怎么知道访问了两个不同的地址的项目呢。
    负载均衡使用的是 ip，而IP对应的服务器使用 相同的访问地址。
    如果想在把负载均衡的那台服务器的资源也利用上可以在 IP地址后台加上其他的端口。如：

```base
upstream study{
    server 192.168.6.252:81;
    server 192.168.6.252:82;
    server 192.168.10.10:80;
}

server {
    listen 80;
    server_name www.study.me;

    location / {
        proxy_pass http://study;
    }
}

server {
    listen 81;
    server_name www.study.me;
    index index.php index.html;
    root /vagrant/H-ui.admin;
}

server {
    listen 82;
    server_name www.study.me;
    index index.php index.html;
    root /vagrant/test;
}

```

2. 如果我使用了负载均衡，其他的服务器需要买带宽吗？
    阿里云云服务器之间的内网带宽是1G 不收费的

坑：

1. 我使用了 www.study.me 来当代理访问网址。但是一访问这个网址就会跳转到 www.test.me。
    我一直以为是nginx 配置问题。结果是因为浏览器记录了301自动跳转了。因为学习的过程有使用 test.me 来当代理点
    处理：清除浏览数据即可。



