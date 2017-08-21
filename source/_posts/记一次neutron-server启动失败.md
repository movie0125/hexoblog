
# 记一次neutron-server启动失败--lbaas驱动加载失败

> 问题现象

neutron-server 启动失败，查看日志，发现是加载 lbaas 驱动失败。
```
2016-12-27 09:17:17.171 76972 CRITICAL neutron [req-b95ec348-cf8f-4381-83d0-e1447825b2aa ] AttributeError: 'NoneType' object has no attribute 'provider_name'
2016-12-27 09:17:17.171 76972 TRACE neutron Traceback (most recent call last):
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/bin/neutron-server", line 10, in <module>
2016-12-27 09:17:17.171 76972 TRACE neutron     sys.exit(main())
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/neutron/cmd/eventlet/server/__init__.py", line 17, in main
2016-12-27 09:17:17.171 76972 TRACE neutron     server.main()
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/neutron/server/__init__.py", line 43, in main
2016-12-27 09:17:17.171 76972 TRACE neutron     neutron_api = service.serve_wsgi(service.NeutronApiService)
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/neutron/service.py", line 106, in serve_wsgi
2016-12-27 09:17:17.171 76972 TRACE neutron     LOG.exception(_LE('Unrecoverable error: please check log '
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/oslo_utils/excutils.py", line 85, in __exit__
2016-12-27 09:17:17.171 76972 TRACE neutron     six.reraise(self.type_, self.value, self.tb)
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/neutron/service.py", line 103, in serve_wsgi
2016-12-27 09:17:17.171 76972 TRACE neutron     service.start()
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/neutron/service.py", line 74, in start
2016-12-27 09:17:17.171 76972 TRACE neutron     self.wsgi_app = _run_wsgi(self.app_name)
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/neutron/service.py", line 169, in _run_wsgi
2016-12-27 09:17:17.171 76972 TRACE neutron     app = config.load_paste_app(app_name)
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/neutron/common/config.py", line 229, in load_paste_app
2016-12-27 09:17:17.171 76972 TRACE neutron     app = deploy.loadapp("config:%s" % config_path, name=app_name)
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/paste/deploy/loadwsgi.py", line 247, in loadapp
2016-12-27 09:17:17.171 76972 TRACE neutron     return loadobj(APP, uri, name=name, **kw)
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/paste/deploy/loadwsgi.py", line 272, in loadobj
2016-12-27 09:17:17.171 76972 TRACE neutron     return context.create()
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/paste/deploy/loadwsgi.py", line 710, in create
2016-12-27 09:17:17.171 76972 TRACE neutron     return self.object_type.invoke(self)
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/paste/deploy/loadwsgi.py", line 144, in invoke
2016-12-27 09:17:17.171 76972 TRACE neutron     **context.local_conf)
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/paste/deploy/util.py", line 56, in fix_call
2016-12-27 09:17:17.171 76972 TRACE neutron     val = callable(*args, **kw)
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/paste/urlmap.py", line 25, in urlmap_factory
2016-12-27 09:17:17.171 76972 TRACE neutron     app = loader.get_app(app_name, global_conf=global_conf)
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/paste/deploy/loadwsgi.py", line 350, in get_app
2016-12-27 09:17:17.171 76972 TRACE neutron     name=name, global_conf=global_conf).create()
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/paste/deploy/loadwsgi.py", line 710, in create
2016-12-27 09:17:17.171 76972 TRACE neutron     return self.object_type.invoke(self)
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/paste/deploy/loadwsgi.py", line 144, in invoke
2016-12-27 09:17:17.171 76972 TRACE neutron     **context.local_conf)
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/paste/deploy/util.py", line 56, in fix_call
2016-12-27 09:17:17.171 76972 TRACE neutron     val = callable(*args, **kw)
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/neutron/auth.py", line 74, in pipeline_factory
2016-12-27 09:17:17.171 76972 TRACE neutron     app = loader.get_app(pipeline[-1])
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/paste/deploy/loadwsgi.py", line 350, in get_app
2016-12-27 09:17:17.171 76972 TRACE neutron     name=name, global_conf=global_conf).create()
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/paste/deploy/loadwsgi.py", line 710, in create
2016-12-27 09:17:17.171 76972 TRACE neutron     return self.object_type.invoke(self)
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/paste/deploy/loadwsgi.py", line 146, in invoke
2016-12-27 09:17:17.171 76972 TRACE neutron     return fix_call(context.object, context.global_conf, **context.local_conf)
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/paste/deploy/util.py", line 56, in fix_call
2016-12-27 09:17:17.171 76972 TRACE neutron     val = callable(*args, **kw)
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/neutron/api/v2/router.py", line 71, in factory
2016-12-27 09:17:17.171 76972 TRACE neutron     return cls(**local_config)
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/neutron/api/v2/router.py", line 75, in __init__
2016-12-27 09:17:17.171 76972 TRACE neutron     plugin = manager.NeutronManager.get_plugin()
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/neutron/manager.py", line 245, in get_plugin
2016-12-27 09:17:17.171 76972 TRACE neutron     return weakref.proxy(cls.get_instance().plugin)
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/neutron/manager.py", line 239, in get_instance
2016-12-27 09:17:17.171 76972 TRACE neutron     cls._create_instance()
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/oslo_concurrency/lockutils.py", line 445, in inner
2016-12-27 09:17:17.171 76972 TRACE neutron     return f(*args, **kwargs)
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/neutron/manager.py", line 225, in _create_instance
2016-12-27 09:17:17.171 76972 TRACE neutron     cls._instance = cls()
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/neutron/manager.py", line 129, in __init__
2016-12-27 09:17:17.171 76972 TRACE neutron     self._load_service_plugins()
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/neutron/manager.py", line 198, in _load_service_plugins
2016-12-27 09:17:17.171 76972 TRACE neutron     provider)
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/neutron/manager.py", line 166, in _get_plugin_instance
2016-12-27 09:17:17.171 76972 TRACE neutron     return plugin_class()
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/neutron_lbaas/services/loadbalancer/plugin.py", line 389, in __init__
2016-12-27 09:17:17.171 76972 TRACE neutron     self._load_drivers()
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/neutron_lbaas/services/loadbalancer/plugin.py", line 405, in _load_drivers
2016-12-27 09:17:17.171 76972 TRACE neutron     self._check_orphan_loadbalancer_associations(ctx, self.drivers.keys())
2016-12-27 09:17:17.171 76972 TRACE neutron   File "/usr/lib/python2.7/site-packages/neutron_lbaas/services/loadbalancer/plugin.py", line 418, in _check_orphan_loadbalancer_associations
2016-12-27 09:17:17.171 76972 TRACE neutron     if lb.provider.provider_name not in provider_names])
2016-12-27 09:17:17.171 76972 TRACE neutron AttributeError: 'NoneType' object has no attribute 'provider_name'
```

> 定位分析

- 查找lbaas配置文见中 驱动配置部分

```python
service_provider = LOADBALANCERV2:Haproxy:neutron_lbaas.drivers.haproxy.plugin_driver.HaproxyOnHostPluginDriver:default
```

- 查看lb驱动代码文件
  
    路径 `/usr/lib/python2.7/site-packages/neutron_lbaas/drivers/haproxy/plugin_driver.py` 
    对比文件路径以及代码与配置文件中值的区别，没有什么问题

- 开发查找代码原因

    发现 lb 启动的时候，会检查现所有 lb 资源的状态，在停止服务时，有一个lb状态为创建中
    重启 neutron-server 会报错，需要清理异常状态的lb。

    这个问题不仅仅是创建的LB资源，删除时也会存在同样的问题。

    代码文件 `/usr/lib/python2.7/site-packages/neutron_lbaas/services/loadbalancer/plugin.py` 405行
```
    def _load_drivers(self):
        """Loads plugin-drivers specified in configuration."""
        self.drivers, self.default_provider = service_base.load_drivers(
            constants.LOADBALANCERV2, self)

        # NOTE(blogan): this method MUST be called after
        # service_base.load_drivers to correctly verify
        verify_lbaas_mutual_exclusion()

        # we're at the point when extensions are not loaded yet
        # so prevent policy from being loaded
        ctx = ncontext.get_admin_context(load_admin_roles=False)
        # stop service in case provider was removed, but resources were not
        self._check_orphan_loadbalancer_associations(ctx, self.drivers.keys())

```
    
> 问题解决

    删除异常状态的lbaas资源

    命令列表
```
  lbaas-loadbalancer-create         LBaaS v2 Create a loadbalancer.
  lbaas-loadbalancer-delete         LBaaS v2 Delete a given loadbalancer.
  lbaas-loadbalancer-list           LBaaS v2 List loadbalancers that belong to a given tenant.
  lbaas-loadbalancer-list-on-agent  List the loadbalancers on a loadbalancer v2 agent.
  lbaas-loadbalancer-show           LBaaS v2 Show information of a given loadbalancer.
  lbaas-loadbalancer-update         LBaaS v2 Update a given loadbalancer.
```

> 问题优化

    根本原因还是在于没有解耦，创建 lbaas 资源异常只是影响一个或部分用户，  
    不应该导致整个 `neutron-server` 服务无法启动。

代码优化的建议：
- 是否考虑启动服务的时候，不去检查异常的资源状态？
- 如果一定需要检查，是否能输出更明确的日志？因为运维看到这个日志，不知道是资源异常引起的。

> 扩展阅读
    
Lbass 资源创建后会生成一个Haproxy provider drivers 与资源一一对应。  
当资源创建或者删除时，可能出现一些意外情况，导致这种对应关系没有建立，  
服务重启时，加载驱动时被检查到了，就异常退出了。

- [OpenStack Neutron 之 Load Balance](http://www.ibm.com/developerworks/cn/cloud/library/1506_xiaohh_openstacklb/)
- [官网Load Balancer as a Service (LBaaS)](http://docs.openstack.org/mitaka/networking-guide/config-lbaas.html)

