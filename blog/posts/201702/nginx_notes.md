<!--
author: checkking
date: 2017-02-24
title: nginx源码阅读点滴
tags: nginx
category: nginx
status: publish
summary: nginx源码阅读
-->
#### ngx_add_inherited_sockets
这个函数的目的是为了实现nginx平滑升级时获取原来的监听fd, 通过环境变量NGINX完成socket的继承，继承来的socket将会放到init_cycle的listening数组中。在NGINX环境变量中，每个socket中间用冒号或分号隔开。完成继承同时设置全局变量ngx_inherited为1。
相关代码：

- src/core/nginx.c
```c
static ngx_int_t
ngx_add_inherited_sockets(ngx_cycle_t *cycle)
{
    u_char           *p, *v, *inherited;
    ngx_int_t         s;
    ngx_listening_t  *ls;

    inherited = (u_char *) getenv(NGINX_VAR);

    if (inherited == NULL) {
        return NGX_OK;
    }

    ngx_log_error(NGX_LOG_NOTICE, cycle->log, 0,
                  "using inherited sockets from \"%s\"", inherited);

    if (ngx_array_init(&cycle->listening, cycle->pool, 10,
                       sizeof(ngx_listening_t))
        != NGX_OK)
    {
        return NGX_ERROR;
    }

    for (p = inherited, v = p; *p; p++) {
        if (*p == ':' || *p == ';') {
            s = ngx_atoi(v, p - v);
            if (s == NGX_ERROR) {
                ngx_log_error(NGX_LOG_EMERG, cycle->log, 0,
                              "invalid socket number \"%s\" in " NGINX_VAR
                              " environment variable, ignoring the rest"
                              " of the variable", v);
                break;
            }

            v = p + 1;

            ls = ngx_array_push(&cycle->listening);
            if (ls == NULL) {
                return NGX_ERROR;
            }

            ngx_memzero(ls, sizeof(ngx_listening_t));

            ls->fd = (ngx_socket_t) s;
        }
    }

    ngx_inherited = 1;

    return ngx_set_inherited_sockets(cycle);
}
```

- src/core/nginx.c:ngx_exec_new_binary
```c
p = ngx_cpymem(var, NGINX_VAR "=", sizeof(NGINX_VAR));
ls = cycle->listening.elts;
for (i = 0; i < cycle->listening.nelts; i++) {
    p = ngx_sprintf(p, "%ud;", ls[i].fd);
}
*p = '\0';
env[n++] = var;
```
