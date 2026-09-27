<template>
  <div class="p-4">
    <div class="md:flex">
      <template v-for="(item, index) in growCardList" :key="item.title">
        <Card
          size="small"
          :loading="loading"
          :title="item.title"
          class="md:w-1/4 w-full !md:mt-0 !mt-4"
          :class="[index + 1 < 4 && '!md:mr-4']"
          :canExpan="false"
        >
          <template #extra>
            <Tag color="green">{{ item.action }}</Tag>
          </template>

          <div class="py-4 px-4 flex justify-between">
            <CountTo prefix="" :startVal="1" :endVal="item.value" class="text-2xl" />
            <Icon :icon="item.icon" :size="40" />
          </div>

          <div class="p-2 px-4 flex justify-between">
            <span>今日已处理</span>
            <CountTo prefix="" :startVal="1" :endVal="item.total" />
          </div>
        </Card>
      </template>

      <template v-for="(item, index) in growCardList2" :key="item.title">
        <Card
          size="small"
          :loading="loading"
          :title="item.title"
          class="md:w-1/4 w-full !md:mt-0 !mt-4"
          :class="[index + 1 < 4 && '!md:mr-4']"
          :canExpan="false"
        >
          <template #extra>
            <Tag color="green">{{ item.action }}</Tag>
          </template>

          <div class="py-4 px-4 flex justify-between">
            <CountTo prefix="" :startVal="1" :endVal="item.value" class="text-2xl" />
            <Icon :icon="item.icon" :size="40" />
          </div>

          <div class="p-2 px-4 flex justify-between">
            <span>今日已处理</span>
            <CountTo prefix="" :startVal="1" :endVal="item.total" />
          </div>
        </Card>
      </template>
    </div>

    <a-card :loading="loading" :bordered="false" :body-style="{ padding: '0' }">
      <div class="salesCard">
        <a-tabs default-active-key="1" size="large" :tab-bar-style="{ marginBottom: '24px', paddingLeft: '16px' }">
          <a-tab-pane loading="true" tab="入库趋势" key="1">
            <a-row>
              <a-col :xl="16" :lg="12" :md="12" :sm="24" :xs="24">
                <Bar
                  :chartData="inOrderBarData"
                  :option="{ title: { text: '', textStyle: { fontWeight: 'lighter' } } }"
                  height="40vh"
                  :seriesColor="seriesColor"
                />
              </a-col>
              <a-col :xl="8" :lg="12" :md="12" :sm="24" :xs="24">
                <RankList title="货主出库排行榜" :list="rankList" />
              </a-col>
            </a-row>
          </a-tab-pane>
          <a-tab-pane tab="出库趋势" key="2">
            <a-row>
              <a-col :xl="16" :lg="12" :md="12" :sm="24" :xs="24">
                <Bar
                  :chartData="outOrderBarData"
                  :option="{ title: { text: '', textStyle: { fontWeight: 'lighter' } } }"
                  height="40vh"
                  :seriesColor="seriesColor"
                />
              </a-col>
              <a-col :xl="8" :lg="12" :md="12" :sm="24" :xs="24">
                <RankList title="门店销售排行榜" :list="rankList" />
              </a-col>
            </a-row>
          </a-tab-pane>
        </a-tabs>
      </div>
    </a-card>

    <a-row>
      <a-col :span="24">
        <a-card :loading="loading" :class="{ 'anty-list-cust': true }" :bordered="false">
          <a-tabs v-model:activeKey="indexBottomTab" size="large" :tab-bar-style="{ marginBottom: '24px', paddingLeft: '16px' }">
            <a-tab-pane tab="库存预警" key="1">
              <a-table
                :dataSource="dataSource"
                size="default"
                rowKey="reBizCode"
                :columns="table.columns"
                :pagination="ipagination"
                @change="tableChange"
              >
              </a-table>
            </a-tab-pane>
          </a-tabs>
        </a-card>
      </a-col>
    </a-row>
  </div>
</template>
<script lang="ts" setup>
  import { ref, watch, computed, unref } from 'vue';
  import { useRootSetting } from '/@/hooks/setting/useRootSetting';
  import { CountTo } from '/@/components/CountTo/index';
  import { Icon } from '/@/components/Icon';
  import { Tag, Card } from 'ant-design-vue';

  import Bar from '/@/components/chart/Bar.vue';
  import RankList from '/@/components/chart/RankList.vue';
  import { defHttp } from '@/utils/http/axios';

  const loading = ref(true);
  const { getThemeColor } = useRootSetting();
  interface GrowCardItem {
    icon: string;
    title: string;
    value?: number;
    total: number;
    color?: string;
    action?: string;
    footer?: string;
  }
  const growCardList = ref([
    {
      title: '待收货任务',
      icon: 'visit-count|svg',
      value: 0,
      total: 0,
      action: '当天',
    },
    {
      title: '待上架任务',
      icon: 'total-sales|svg',
      value: 0,
      total: 0,
      action: '当天',
    },
    {
      title: '待拣货任务',
      icon: 'download-count|svg',
      value: 0,
      total: 0,
      action: '当天',
    },
  ]);
  const growCardList2 = ref([
    {
      title: '待打包数',
      icon: 'transaction|svg',
      value: 0,
      total: 0,
      action: '当天',
    },
  ]);

  const table = {
    dataSource: [
      { reBizCode: '1', type: '转移登记', acceptBy: '张三', acceptDate: '2019-01-22', curNode: '任务分派', flowRate: 60 },
      { reBizCode: '2', type: '抵押登记', acceptBy: '李四', acceptDate: '2019-01-23', curNode: '领导审核', flowRate: 30 },
      { reBizCode: '3', type: '转移登记', acceptBy: '王武', acceptDate: '2019-01-25', curNode: '任务处理', flowRate: 20 },
      { reBizCode: '4', type: '转移登记', acceptBy: '赵楼', acceptDate: '2019-11-22', curNode: '部门审核', flowRate: 80 },
      { reBizCode: '5', type: '转移登记', acceptBy: '钱就', acceptDate: '2019-12-12', curNode: '任务分派', flowRate: 90 },
      { reBizCode: '6', type: '转移登记', acceptBy: '孙吧', acceptDate: '2019-03-06', curNode: '任务处理', flowRate: 10 },
      { reBizCode: '7', type: '抵押登记', acceptBy: '周大', acceptDate: '2019-04-13', curNode: '任务分派', flowRate: 100 },
      { reBizCode: '8', type: '抵押登记', acceptBy: '吴二', acceptDate: '2019-05-09', curNode: '任务上报', flowRate: 50 },
      { reBizCode: '9', type: '抵押登记', acceptBy: '郑爽', acceptDate: '2019-07-12', curNode: '任务处理', flowRate: 63 },
      { reBizCode: '20', type: '抵押登记', acceptBy: '林有', acceptDate: '2019-12-12', curNode: '任务打回', flowRate: 59 },
      { reBizCode: '11', type: '转移登记', acceptBy: '码云', acceptDate: '2019-09-10', curNode: '任务签收', flowRate: 87 },
    ],
    columns: [
      {
        title: '业务号',
        align: 'center',
        dataIndex: 'reBizCode',
      },
      {
        title: '业务类型',
        align: 'center',
        dataIndex: 'type',
      },
      {
        title: '受理人',
        align: 'center',
        dataIndex: 'acceptBy',
      },
      {
        title: '受理时间',
        align: 'center',
        dataIndex: 'acceptDate',
      },
      {
        title: '当前节点',
        align: 'center',
        dataIndex: 'curNode',
      },
      {
        title: '办理时长',
        align: 'center',
        dataIndex: 'flowRate',
      },
    ],
    ipagination: {
      current: 1,
      pageSize: 5,
      pageSizeOptions: ['10', '20', '30'],
      showTotal: (total, range) => {
        return range[0] + '-' + range[1] + ' 共' + total + '条';
      },
      showQuickJumper: true,
      showSizeChanger: true,
      total: 0,
    },
  };

  defineProps({
    loading: {
      type: Boolean,
    },
  });
  const rankList = ref([
    {
      name: '宝可梦 1号店',
      total: 1234.56,
    },
    {
      name: 'E:\\jdk_17\\bin\\java.exe -XX:TieredStopAtLevel=1 -Dspring.output.ansi.enabled=always -Dcom.sun.management.jmxremote -Dspring.jmx.enabled=true -Dspring.liveBeansView.mbeanDomain -Dspring.application.admin.enabled=true "-Dmanagement.endpoints.jmx.exposure.include=*" "-javaagent:E:\\java\\IntelliJ IDEA 2025.3.4\\lib\\idea_rt.jar=53953" -Dfile.encoding=UTF-8 -classpath C:\\Users\\Administrator\\AppData\\Local\\Temp\\classpath1910347642.jar org.jeecg.JeecgSystemApplication\n'+
        'Standard Commons Logging discovery in action with spring-jcl: please remove commons-logging.jar from classpath in order to avoid potential conflicts\n'+
        '15:28:45.477 [main] INFO org.jeecg.JeecgSystemApplication -- [JEECG] Elasticsearch Health Check Enabled: false\n'+
        '\n'+
        '   (_)                          | |               | |\n'+
        '    _  ___  ___  ___ __ _ ______| |__   ___   ___ | |_ \n'+
        '   | |/ _ \\/ _ \\/ __/ _` |______| \'_ \\ / _ \\ / _ \\| __|\n'+
        '   | |  __/  __/ (_| (_| |      | |_) | (_) | (_) | |_ \n'+
        '   | |\\___|\\___|\\___\\__, |      |_.__/ \\___/ \\___/ \\__|\n'+
        '  _/ |               __/ |                             \n'+
        ' |__/               |___/\n'+
        '\n'+
        '\n'+
        '\n'+
        'Jeecg  Boot Version: 3.8.1\n'+
        'Spring Boot Version: 3.4.5 (v3.4.5)\n'+
        '产品官网： www.jeecg.com\n'+
        '版权所属： 北京国炬信息技术有限公司\n'+
        '公司官网： www.guojusoft.com\n'+
        '\n'+
        '\n'+
        '2026-09-18 15:28:46.787 [background-preinit] INFO  org.hibernate.validator.internal.util.Version:21 - HV000001: Hibernate Validator 8.0.2.Final\n'+
        '2026-09-18 15:28:46.948 [main] INFO  org.jeecg.JeecgSystemApplication:53 - Starting JeecgSystemApplication using Java 17.0.19 with PID 12388 (E:\\星辰下载\\【项目课程】星辰WMS\\代码\\day16 IOT-核心业务\\代码\\xincheng-wms-java-course\\jeecg-module-system\\jeecg-system-start\\target\\classes started by Administrator in E:\\星辰下载\\【项目课程】星辰WMS\\代码\\day16 IOT-核心业务\\代码\\xincheng-wms-java-course)\n'+
        '2026-09-18 15:28:46.951 [main] INFO  org.jeecg.JeecgSystemApplication:658 - The following 1 profile is active: "dev"\n'+
        '2026-09-18 15:28:51.486 [main] INFO  o.s.d.r.config.RepositoryConfigurationDelegate:295 - Multiple Spring Data modules found, entering strict repository configuration mode\n'+
        '2026-09-18 15:28:51.491 [main] INFO  o.s.d.r.config.RepositoryConfigurationDelegate:143 - Bootstrapping Spring Data Redis repositories in DEFAULT mode.\n'+
        '2026-09-18 15:28:51.870 [main] INFO  o.s.d.r.config.RepositoryConfigurationDelegate:211 - Finished Spring Data repository scanning in 354 ms. Found 0 Redis repository interfaces.\n'+
        '2026-09-18 15:28:52.015 [main] INFO  o.j.minidao.auto.MinidaoAutoConfiguration:23 -  ******************* init miniDao config [ begin ] *********************** \n'+
        '2026-09-18 15:28:52.016 [main] INFO  o.j.minidao.auto.MinidaoAutoConfiguration:25 -  ------ minidao.base-package ------- org.jeecg.modules.jmreport.*,org.jeecg.modules.drag.*\n'+
        '2026-09-18 15:28:52.018 [main] INFO  o.j.minidao.auto.MinidaoAutoConfiguration:42 -  *******************  init miniDao config  [ end ] *********************** \n'+
        '2026-09-18 15:28:52.662 [main] INFO  o.s.b.factory.support.DefaultListableBeanFactory:1333 - Overriding bean definition for bean \'classifier\' with a different definition: replacing [Generic bean: class=org.jeecg.modules.airag.flow.component.a; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; fallback=false; factoryBeanName=null; factoryMethodName=null; initMethodNames=null; destroyMethodNames=null; defined in URL [jar:file:/E:/maven/apache-maven-3.9.4-bin/apache-maven-3.9.4/mvn_repo/org/jeecgframework/boot3/jeecg-aiflow/1.0.5/jeecg-aiflow-1.0.5.jar!/org/jeecg/modules/airag/flow/component/a.class]] with [Generic bean: class=com.yomahub.liteflow.core.proxy.DeclWarpBean; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; fallback=false; factoryBeanName=null; factoryMethodName=null; initMethodNames=null; destroyMethodNames=null]\n'+
        '2026-09-18 15:28:52.664 [main] INFO  o.s.b.factory.support.DefaultListableBeanFactory:1333 - Overriding bean definition for bean \'end\' with a different definition: replacing [Generic bean: class=org.jeecg.modules.airag.flow.component.b; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; fallback=false; factoryBeanName=null; factoryMethodName=null; initMethodNames=null; destroyMethodNames=null; defined in URL [jar:file:/E:/maven/apache-maven-3.9.4-bin/apache-maven-3.9.4/mvn_repo/org/jeecgframework/boot3/jeecg-aiflow/1.0.5/jeecg-aiflow-1.0.5.jar!/org/jeecg/modules/airag/flow/component/b.class]] with [Generic bean: class=com.yomahub.liteflow.core.proxy.DeclWarpBean; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; fallback=false; factoryBeanName=null; factoryMethodName=null; initMethodNames=null; destroyMethodNames=null]\n'+
        '2026-09-18 15:28:52.665 [main] INFO  o.s.b.factory.support.DefaultListableBeanFactory:1333 - Overriding bean definition for bean \'http\' with a different definition: replacing [Generic bean: class=org.jeecg.modules.airag.flow.component.c; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; fallback=false; factoryBeanName=null; factoryMethodName=null; initMethodNames=null; destroyMethodNames=null; defined in URL [jar:file:/E:/maven/apache-maven-3.9.4-bin/apache-maven-3.9.4/mvn_repo/org/jeecgframework/boot3/jeecg-aiflow/1.0.5/jeecg-aiflow-1.0.5.jar!/org/jeecg/modules/airag/flow/component/c.class]] with [Generic bean: class=com.yomahub.liteflow.core.proxy.DeclWarpBean; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; fallback=false; factoryBeanName=null; factoryMethodName=null; initMethodNames=null; destroyMethodNames=null]\n'+
        '2026-09-18 15:28:52.666 [main] INFO  o.s.b.factory.support.DefaultListableBeanFactory:1333 - Overriding bean definition for bean \'knowledge\' with a different definition: replacing [Generic bean: class=org.jeecg.modules.airag.flow.component.d; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; fallback=false; factoryBeanName=null; factoryMethodName=null; initMethodNames=null; destroyMethodNames=null; defined in URL [jar:file:/E:/maven/apache-maven-3.9.4-bin/apache-maven-3.9.4/mvn_repo/org/jeecgframework/boot3/jeecg-aiflow/1.0.5/jeecg-aiflow-1.0.5.jar!/org/jeecg/modules/airag/flow/component/d.class]] with [Generic bean: class=com.yomahub.liteflow.core.proxy.DeclWarpBean; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; fallback=false; factoryBeanName=null; factoryMethodName=null; initMethodNames=null; destroyMethodNames=null]\n'+
        '2026-09-18 15:28:52.666 [main] INFO  o.s.b.factory.support.DefaultListableBeanFactory:1333 - Overriding bean definition for bean \'llm\' with a different definition: replacing [Generic bean: class=org.jeecg.modules.airag.flow.component.e; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; fallback=false; factoryBeanName=null; factoryMethodName=null; initMethodNames=null; destroyMethodNames=null; defined in URL [jar:file:/E:/maven/apache-maven-3.9.4-bin/apache-maven-3.9.4/mvn_repo/org/jeecgframework/boot3/jeecg-aiflow/1.0.5/jeecg-aiflow-1.0.5.jar!/org/jeecg/modules/airag/flow/component/e.class]] with [Generic bean: class=com.yomahub.liteflow.core.proxy.DeclWarpBean; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; fallback=false; factoryBeanName=null; factoryMethodName=null; initMethodNames=null; destroyMethodNames=null]\n'+
        '2026-09-18 15:28:52.667 [main] INFO  o.s.b.factory.support.DefaultListableBeanFactory:1333 - Overriding bean definition for bean \'enhanceJava\' with a different definition: replacing [Generic bean: class=org.jeecg.modules.airag.flow.component.enhance.a; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; fallback=false; factoryBeanName=null; factoryMethodName=null; initMethodNames=null; destroyMethodNames=null; defined in URL [jar:file:/E:/maven/apache-maven-3.9.4-bin/apache-maven-3.9.4/mvn_repo/org/jeecgframework/boot3/jeecg-aiflow/1.0.5/jeecg-aiflow-1.0.5.jar!/org/jeecg/modules/airag/flow/component/enhance/a.class]] with [Generic bean: class=com.yomahub.liteflow.core.proxy.DeclWarpBean; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; fallback=false; factoryBeanName=null; factoryMethodName=null; initMethodNames=null; destroyMethodNames=null]\n'+
        '2026-09-18 15:28:52.669 [main] INFO  o.s.b.factory.support.DefaultListableBeanFactory:1333 - Overriding bean definition for bean \'reply\' with a different definition: replacing [Generic bean: class=org.jeecg.modules.airag.flow.component.f; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; fallback=false; factoryBeanName=null; factoryMethodName=null; initMethodNames=null; destroyMethodNames=null; defined in URL [jar:file:/E:/maven/apache-maven-3.9.4-bin/apache-maven-3.9.4/mvn_repo/org/jeecgframework/boot3/jeecg-aiflow/1.0.5/jeecg-aiflow-1.0.5.jar!/org/jeecg/modules/airag/flow/component/f.class]] with [Generic bean: class=com.yomahub.liteflow.core.proxy.DeclWarpBean; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; fallback=false; factoryBeanName=null; factoryMethodName=null; initMethodNames=null; destroyMethodNames=null]\n'+
        '2026-09-18 15:28:52.669 [main] INFO  o.s.b.factory.support.DefaultListableBeanFactory:1333 - Overriding bean definition for bean \'start\' with a different definition: replacing [Generic bean: class=org.jeecg.modules.airag.flow.component.g; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; fallback=false; factoryBeanName=null; factoryMethodName=null; initMethodNames=null; destroyMethodNames=null; defined in URL [jar:file:/E:/maven/apache-maven-3.9.4-bin/apache-maven-3.9.4/mvn_repo/org/jeecgframework/boot3/jeecg-aiflow/1.0.5/jeecg-aiflow-1.0.5.jar!/org/jeecg/modules/airag/flow/component/g.class]] with [Generic bean: class=com.yomahub.liteflow.core.proxy.DeclWarpBean; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; fallback=false; factoryBeanName=null; factoryMethodName=null; initMethodNames=null; destroyMethodNames=null]\n'+
        '2026-09-18 15:28:52.671 [main] INFO  o.s.b.factory.support.DefaultListableBeanFactory:1333 - Overriding bean definition for bean \'subflow\' with a different definition: replacing [Generic bean: class=org.jeecg.modules.airag.flow.component.h; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; fallback=false; factoryBeanName=null; factoryMethodName=null; initMethodNames=null; destroyMethodNames=null; defined in URL [jar:file:/E:/maven/apache-maven-3.9.4-bin/apache-maven-3.9.4/mvn_repo/org/jeecgframework/boot3/jeecg-aiflow/1.0.5/jeecg-aiflow-1.0.5.jar!/org/jeecg/modules/airag/flow/component/h.class]] with [Generic bean: class=com.yomahub.liteflow.core.proxy.DeclWarpBean; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; fallback=false; factoryBeanName=null; factoryMethodName=null; initMethodNames=null; destroyMethodNames=null]\n'+
        '2026-09-18 15:28:52.672 [main] INFO  o.s.b.factory.support.DefaultListableBeanFactory:1333 - Overriding bean definition for bean \'switch\' with a different definition: replacing [Generic bean: class=org.jeecg.modules.airag.flow.component.i; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; fallback=false; factoryBeanName=null; factoryMethodName=null; initMethodNames=null; destroyMethodNames=null; defined in URL [jar:file:/E:/maven/apache-maven-3.9.4-bin/apache-maven-3.9.4/mvn_repo/org/jeecgframework/boot3/jeecg-aiflow/1.0.5/jeecg-aiflow-1.0.5.jar!/org/jeecg/modules/airag/flow/component/i.class]] with [Generic bean: class=com.yomahub.liteflow.core.proxy.DeclWarpBean; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; fallback=false; factoryBeanName=null; factoryMethodName=null; initMethodNames=null; destroyMethodNames=null]\n'+
        '2026-09-18 15:28:53.224 [main] WARN  o.j.minidao.factory.MiniDaoClassPathMapperScanner:42 - No Dao interface was found in \'[org.jeecg.modules.jmreport.*, org.jeecg.modules.drag.*]\' package. Please check your configuration.\n'+
        '2026-09-18 15:28:53.227 [main] WARN  o.s.c.annotation.ConfigurationClassPostProcessor:514 - Cannot enhance @Configuration bean definition \'com.yomahub.liteflow.springboot.config.LiteflowMainAutoConfiguration\' since its singleton instance has been created too early. The typical cause is a non-static @Bean method with a BeanDefinitionRegistryPostProcessor return type: Consider declaring such methods as \'static\' and/or marking the containing configuration class as \'proxyBeanMethods=false\'.\n'+
        '2026-09-18 15:28:54.346 [main] INFO  org.jeecg.config.shiro.ShiroConfig:290 - ===============(1)创建缓存管理器RedisCacheManager\n'+
        '2026-09-18 15:28:54.350 [main] INFO  org.jeecg.config.shiro.ShiroConfig:311 - ===============(2)创建RedisManager,连接Redis..\n'+
        '2026-09-18 15:28:55.606 [main] INFO  io.undertow.servlet:372 - Initializing Spring embedded WebApplicationContext\n'+
        '2026-09-18 15:28:55.606 [main] INFO  o.s.b.w.s.c.ServletWebServerApplicationContext:301 - Root WebApplicationContext: initialization completed in 8053 ms\n'+
        '2026-09-18 15:28:56.619 [main] INFO  com.alibaba.druid.pool.DruidDataSource:673 - {dataSource-1,mysql} inited\n'+
        '2026-09-18 15:28:56.672 [main] ERROR com.alibaba.druid.pool.DruidDataSource:599 - init datasource error, url: jdbc:postgresql://127.0.0.1:5432/postgres\n'+
        'org.postgresql.util.PSQLException: Connection to 127.0.0.1:5432 refused. Check that the hostname and port are correct and that the postmaster is accepting TCP/IP connections.\n'+
        '\tat org.postgresql.core.v3.ConnectionFactoryImpl.openConnectionImpl(ConnectionFactoryImpl.java:303)\n'+
        '\tat org.postgresql.core.ConnectionFactory.openConnection(ConnectionFactory.java:51)\n'+
        '\tat org.postgresql.jdbc.PgConnection.<init>(PgConnection.java:225)\n'+
        '\tat org.postgresql.Driver.makeConnection(Driver.java:465)\n'+
        '\tat org.postgresql.Driver.connect(Driver.java:264)\n'+
        '\tat com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:132)\n'+
        '\tat com.alibaba.druid.filter.FilterAdapter.connection_connect(FilterAdapter.java:764)\n'+
        '\tat com.alibaba.druid.filter.FilterEventAdapter.connection_connect(FilterEventAdapter.java:33)\n'+
        '\tat com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:126)\n'+
        '\tat com.alibaba.druid.filter.stat.StatFilter.connection_connect(StatFilter.java:244)\n'+
        '\tat com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:126)\n'+
        '\tat com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1687)\n'+
        '\tat com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1803)\n'+
        '\tat com.alibaba.druid.pool.DruidDataSource.init(DruidDataSource.java:595)\n'+
        '\tat com.baomidou.dynamic.datasource.creator.druid.DruidDataSourceCreator.createDataSource(DruidDataSourceCreator.java:132)\n'+
        '\tat com.baomidou.dynamic.datasource.creator.DefaultDataSourceCreator.createDataSource(DefaultDataSourceCreator.java:97)\n'+
        '\tat com.baomidou.dynamic.datasource.provider.AbstractDataSourceProvider.createDataSourceMap(AbstractDataSourceProvider.java:53)\n'+
        '\tat com.baomidou.dynamic.datasource.provider.YmlDynamicDataSourceProvider.loadDataSources(YmlDynamicDataSourceProvider.java:53)\n'+
        '\tat com.baomidou.dynamic.datasource.DynamicRoutingDataSource.afterPropertiesSet(DynamicRoutingDataSource.java:225)\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1865)\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1814)\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:607)\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:529)\n'+
        '\tat org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:339)\n'+
        '\tat org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:371)\n'+
        '\tat org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:337)\n'+
        '\tat org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:202)\n'+
        '\tat org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1681)\n'+
        '\tat org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1627)\n'+
        '\tat org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.resolveFieldValue(AutowiredAnnotationBeanPostProcessor.java:785)\n'+
        '\tat org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:768)\n'+
        '\tat org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:146)\n'+
        '\tat org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessProperties(AutowiredAnnotationBeanPostProcessor.java:509)\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1451)\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:606)\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:529)\n'+
        '\tat org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:339)\n'+
        '\tat org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:371)\n'+
        '\tat org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:337)\n'+
        '\tat org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:202)\n'+
        '\tat org.springframework.beans.factory.support.DefaultListableBeanFactory.instantiateSingleton(DefaultListableBeanFactory.java:1221)\n'+
        '\tat org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingleton(DefaultListableBeanFactory.java:1187)\n'+
        '\tat org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:1122)\n'+
        '\tat org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:987)\n'+
        '\tat org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:627)\n'+
        '\tat org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:146)\n'+
        '\tat org.springframework.boot.SpringApplication.refresh(SpringApplication.java:753)\n'+
        '\tat org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:439)\n'+
        '\tat org.springframework.boot.SpringApplication.run(SpringApplication.java:318)\n'+
        '\tat org.jeecg.JeecgSystemApplication.main(JeecgSystemApplication.java:44)\n'+
        'Caused by: java.net.ConnectException: Connection refused: getsockopt\n'+
        '\tat java.base/sun.nio.ch.Net.pollConnect(Native Method)\n'+
        '\tat java.base/sun.nio.ch.Net.pollConnectNow(Net.java:684)\n'+
        '\tat java.base/sun.nio.ch.NioSocketImpl.timedFinishConnect(NioSocketImpl.java:547)\n'+
        '\tat java.base/sun.nio.ch.NioSocketImpl.connect(NioSocketImpl.java:602)\n'+
        '\tat java.base/java.net.SocksSocketImpl.connect(SocksSocketImpl.java:327)\n'+
        '\tat java.base/java.net.Socket.connect(Socket.java:633)\n'+
        '\tat org.postgresql.core.PGStream.createSocket(PGStream.java:231)\n'+
        '\tat org.postgresql.core.PGStream.<init>(PGStream.java:95)\n'+
        '\tat org.postgresql.core.v3.ConnectionFactoryImpl.tryConnect(ConnectionFactoryImpl.java:98)\n'+
        '\tat org.postgresql.core.v3.ConnectionFactoryImpl.openConnectionImpl(ConnectionFactoryImpl.java:213)\n'+
        '\t... 49 common frames omitted\n'+
        '2026-09-18 15:28:56.678 [main] ERROR com.alibaba.druid.pool.DruidDataSource:648 - {dataSource-2} init error\n'+
        'org.postgresql.util.PSQLException: Connection to 127.0.0.1:5432 refused. Check that the hostname and port are correct and that the postmaster is accepting TCP/IP connections.\n'+
        '\tat org.postgresql.core.v3.ConnectionFactoryImpl.openConnectionImpl(ConnectionFactoryImpl.java:303)\n'+
        '\tat org.postgresql.core.ConnectionFactory.openConnection(ConnectionFactory.java:51)\n'+
        '\tat org.postgresql.jdbc.PgConnection.<init>(PgConnection.java:225)\n'+
        '\tat org.postgresql.Driver.makeConnection(Driver.java:465)\n'+
        '\tat org.postgresql.Driver.connect(Driver.java:264)\n'+
        '\tat com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:132)\n'+
        '\tat com.alibaba.druid.filter.FilterAdapter.connection_connect(FilterAdapter.java:764)\n'+
        '\tat com.alibaba.druid.filter.FilterEventAdapter.connection_connect(FilterEventAdapter.java:33)\n'+
        '\tat com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:126)\n'+
        '\tat com.alibaba.druid.filter.stat.StatFilter.connection_connect(StatFilter.java:244)\n'+
        '\tat com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:126)\n'+
        '\tat com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1687)\n'+
        '\tat com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1803)\n'+
        '\tat com.alibaba.druid.pool.DruidDataSource.init(DruidDataSource.java:595)\n'+
        '\tat com.baomidou.dynamic.datasource.creator.druid.DruidDataSourceCreator.createDataSource(DruidDataSourceCreator.java:132)\n'+
        '\tat com.baomidou.dynamic.datasource.creator.DefaultDataSourceCreator.createDataSource(DefaultDataSourceCreator.java:97)\n'+
        '\tat com.baomidou.dynamic.datasource.provider.AbstractDataSourceProvider.createDataSourceMap(AbstractDataSourceProvider.java:53)\n'+
        '\tat com.baomidou.dynamic.datasource.provider.YmlDynamicDataSourceProvider.loadDataSources(YmlDynamicDataSourceProvider.java:53)\n'+
        '\tat com.baomidou.dynamic.datasource.DynamicRoutingDataSource.afterPropertiesSet(DynamicRoutingDataSource.java:225)\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1865)\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1814)\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:607)\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:529)\n'+
        '\tat org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:339)\n'+
        '\tat org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:371)\n'+
        '\tat org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:337)\n'+
        '\tat org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:202)\n'+
        '\tat org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1681)\n'+
        '\tat org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1627)\n'+
        '\tat org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.resolveFieldValue(AutowiredAnnotationBeanPostProcessor.java:785)\n'+
        '\tat org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:768)\n'+
        '\tat org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:146)\n'+
        '\tat org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessProperties(AutowiredAnnotationBeanPostProcessor.java:509)\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1451)\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:606)\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:529)\n'+
        '\tat org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:339)\n'+
        '\tat org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:371)\n'+
        '\tat org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:337)\n'+
        '\tat org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:202)\n'+
        '\tat org.springframework.beans.factory.support.DefaultListableBeanFactory.instantiateSingleton(DefaultListableBeanFactory.java:1221)\n'+
        '\tat org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingleton(DefaultListableBeanFactory.java:1187)\n'+
        '\tat org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:1122)\n'+
        '\tat org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:987)\n'+
        '\tat org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:627)\n'+
        '\tat org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:146)\n'+
        '\tat org.springframework.boot.SpringApplication.refresh(SpringApplication.java:753)\n'+
        '\tat org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:439)\n'+
        '\tat org.springframework.boot.SpringApplication.run(SpringApplication.java:318)\n'+
        '\tat org.jeecg.JeecgSystemApplication.main(JeecgSystemApplication.java:44)\n'+
        'Caused by: java.net.ConnectException: Connection refused: getsockopt\n'+
        '\tat java.base/sun.nio.ch.Net.pollConnect(Native Method)\n'+
        '\tat java.base/sun.nio.ch.Net.pollConnectNow(Net.java:684)\n'+
        '\tat java.base/sun.nio.ch.NioSocketImpl.timedFinishConnect(NioSocketImpl.java:547)\n'+
        '\tat java.base/sun.nio.ch.NioSocketImpl.connect(NioSocketImpl.java:602)\n'+
        '\tat java.base/java.net.SocksSocketImpl.connect(SocksSocketImpl.java:327)\n'+
        '\tat java.base/java.net.Socket.connect(Socket.java:633)\n'+
        '\tat org.postgresql.core.PGStream.createSocket(PGStream.java:231)\n'+
        '\tat org.postgresql.core.PGStream.<init>(PGStream.java:95)\n'+
        '\tat org.postgresql.core.v3.ConnectionFactoryImpl.tryConnect(ConnectionFactoryImpl.java:98)\n'+
        '\tat org.postgresql.core.v3.ConnectionFactoryImpl.openConnectionImpl(ConnectionFactoryImpl.java:213)\n'+
        '\t... 49 common frames omitted\n'+
        '2026-09-18 15:28:56.679 [main] INFO  com.alibaba.druid.pool.DruidDataSource:673 - {dataSource-2,postgresql} inited\n'+
        '2026-09-18 15:28:56.679 [main] WARN  o.s.b.w.s.c.AnnotationConfigServletWebServerApplicationContext:635 - Exception encountered during context initialization - cancelling refresh attempt: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name \'flywayConfig\': Unsatisfied dependency expressed through field \'dataSource\': Error creating bean with name \'dataSource\' defined in class path resource [com/baomidou/dynamic/datasource/spring/boot/autoconfigure/DynamicDataSourceAutoConfiguration.class]: druid create error\n'+
        '2026-09-18 15:28:56.679 [Druid-ConnectionPool-Create-1924874046] ERROR com.alibaba.druid.pool.DruidDataSource:2592 - create connection SQLException, url: jdbc:postgresql://127.0.0.1:5432/postgres, errorCode 0, state 08001\n'+
        'org.postgresql.util.PSQLException: Connection to 127.0.0.1:5432 refused. Check that the hostname and port are correct and that the postmaster is accepting TCP/IP connections.\n'+
        '\tat org.postgresql.core.v3.ConnectionFactoryImpl.openConnectionImpl(ConnectionFactoryImpl.java:303)\n'+
        '\tat org.postgresql.core.ConnectionFactory.openConnection(ConnectionFactory.java:51)\n'+
        '\tat org.postgresql.jdbc.PgConnection.<init>(PgConnection.java:225)\n'+
        '\tat org.postgresql.Driver.makeConnection(Driver.java:465)\n'+
        '\tat org.postgresql.Driver.connect(Driver.java:264)\n'+
        '\tat com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:132)\n'+
        '\tat com.alibaba.druid.filter.FilterAdapter.connection_connect(FilterAdapter.java:764)\n'+
        '\tat com.alibaba.druid.filter.FilterEventAdapter.connection_connect(FilterEventAdapter.java:33)\n'+
        '\tat com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:126)\n'+
        '\tat com.alibaba.druid.filter.stat.StatFilter.connection_connect(StatFilter.java:244)\n'+
        '\tat com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:126)\n'+
        '\tat com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1687)\n'+
        '\tat com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1803)\n'+
        '\tat com.alibaba.druid.pool.DruidDataSource$CreateConnectionThread.run(DruidDataSource.java:2590)\n'+
        'Caused by: java.net.ConnectException: Connection refused: getsockopt\n'+
        '\tat java.base/sun.nio.ch.Net.pollConnect(Native Method)\n'+
        '\tat java.base/sun.nio.ch.Net.pollConnectNow(Net.java:684)\n'+
        '\tat java.base/sun.nio.ch.NioSocketImpl.timedFinishConnect(NioSocketImpl.java:547)\n'+
        '\tat java.base/sun.nio.ch.NioSocketImpl.connect(NioSocketImpl.java:602)\n'+
        '\tat java.base/java.net.SocksSocketImpl.connect(SocksSocketImpl.java:327)\n'+
        '\tat java.base/java.net.Socket.connect(Socket.java:633)\n'+
        '\tat org.postgresql.core.PGStream.createSocket(PGStream.java:231)\n'+
        '\tat org.postgresql.core.PGStream.<init>(PGStream.java:95)\n'+
        '\tat org.postgresql.core.v3.ConnectionFactoryImpl.tryConnect(ConnectionFactoryImpl.java:98)\n'+
        '\tat org.postgresql.core.v3.ConnectionFactoryImpl.openConnectionImpl(ConnectionFactoryImpl.java:213)\n'+
        '\t... 13 common frames omitted\n'+
        '2026-09-18 15:28:56.681 [Druid-ConnectionPool-Create-1924874046] ERROR com.alibaba.druid.pool.DruidDataSource:2592 - create connection SQLException, url: jdbc:postgresql://127.0.0.1:5432/postgres, errorCode 0, state 08001\n'+
        'org.postgresql.util.PSQLException: Connection to 127.0.0.1:5432 refused. Check that the hostname and port are correct and that the postmaster is accepting TCP/IP connections.\n'+
        '\tat org.postgresql.core.v3.ConnectionFactoryImpl.openConnectionImpl(ConnectionFactoryImpl.java:303)\n'+
        '\tat org.postgresql.core.ConnectionFactory.openConnection(ConnectionFactory.java:51)\n'+
        '\tat org.postgresql.jdbc.PgConnection.<init>(PgConnection.java:225)\n'+
        '\tat org.postgresql.Driver.makeConnection(Driver.java:465)\n'+
        '\tat org.postgresql.Driver.connect(Driver.java:264)\n'+
        '\tat com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:132)\n'+
        '\tat com.alibaba.druid.filter.FilterAdapter.connection_connect(FilterAdapter.java:764)\n'+
        '\tat com.alibaba.druid.filter.FilterEventAdapter.connection_connect(FilterEventAdapter.java:33)\n'+
        '\tat com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:126)\n'+
        '\tat com.alibaba.druid.filter.stat.StatFilter.connection_connect(StatFilter.java:244)\n'+
        '\tat com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:126)\n'+
        '\tat com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1687)\n'+
        '\tat com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1803)\n'+
        '\tat com.alibaba.druid.pool.DruidDataSource$CreateConnectionThread.run(DruidDataSource.java:2590)\n'+
        'Caused by: java.net.ConnectException: Connection refused: getsockopt\n'+
        '\tat java.base/sun.nio.ch.Net.pollConnect(Native Method)\n'+
        '\tat java.base/sun.nio.ch.Net.pollConnectNow(Net.java:684)\n'+
        '\tat java.base/sun.nio.ch.NioSocketImpl.timedFinishConnect(NioSocketImpl.java:547)\n'+
        '\tat java.base/sun.nio.ch.NioSocketImpl.connect(NioSocketImpl.java:602)\n'+
        '\tat java.base/java.net.SocksSocketImpl.connect(SocksSocketImpl.java:327)\n'+
        '\tat java.base/java.net.Socket.connect(Socket.java:633)\n'+
        '\tat org.postgresql.core.PGStream.createSocket(PGStream.java:231)\n'+
        '\tat org.postgresql.core.PGStream.<init>(PGStream.java:95)\n'+
        '\tat org.postgresql.core.v3.ConnectionFactoryImpl.tryConnect(ConnectionFactoryImpl.java:98)\n'+
        '\tat org.postgresql.core.v3.ConnectionFactoryImpl.openConnectionImpl(ConnectionFactoryImpl.java:213)\n'+
        '\t... 13 common frames omitted\n'+
        '2026-09-18 15:28:56.682 [Druid-ConnectionPool-Create-1924874046] INFO  com.alibaba.druid.pool.DruidAbstractDataSource:1900 - {dataSource-2} failContinuous is true\n'+
        '2026-09-18 15:28:56.722 [main] INFO  o.s.b.a.logging.ConditionEvaluationReportLogger:82 - \n'+
        '\n'+
        'Error starting ApplicationContext. To display the condition evaluation report re-run your application with \'debug\' enabled.\n'+
        '2026-09-18 15:28:56.762 [main] ERROR org.springframework.boot.SpringApplication:858 - Application run failed\n'+
        'org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name \'flywayConfig\': Unsatisfied dependency expressed through field \'dataSource\': Error creating bean with name \'dataSource\' defined in class path resource [com/baomidou/dynamic/datasource/spring/boot/autoconfigure/DynamicDataSourceAutoConfiguration.class]: druid create error\n'+
        '\tat org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.resolveFieldValue(AutowiredAnnotationBeanPostProcessor.java:788)\n'+
        '\tat org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:768)\n'+
        '\tat org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:146)\n'+
        '\tat org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessProperties(AutowiredAnnotationBeanPostProcessor.java:509)\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1451)\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:606)\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:529)\n'+
        '\tat org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:339)\n'+
        '\tat org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:371)\n'+
        '\tat org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:337)\n'+
        '\tat org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:202)\n'+
        '\tat org.springframework.beans.factory.support.DefaultListableBeanFactory.instantiateSingleton(DefaultListableBeanFactory.java:1221)\n'+
        '\tat org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingleton(DefaultListableBeanFactory.java:1187)\n'+
        '\tat org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:1122)\n'+
        '\tat org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:987)\n'+
        '\tat org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:627)\n'+
        '\tat org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:146)\n'+
        '\tat org.springframework.boot.SpringApplication.refresh(SpringApplication.java:753)\n'+
        '\tat org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:439)\n'+
        '\tat org.springframework.boot.SpringApplication.run(SpringApplication.java:318)\n'+
        '\tat org.jeecg.JeecgSystemApplication.main(JeecgSystemApplication.java:44)\n'+
        'Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name \'dataSource\' defined in class path resource [com/baomidou/dynamic/datasource/spring/boot/autoconfigure/DynamicDataSourceAutoConfiguration.class]: druid create error\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1818)\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:607)\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:529)\n'+
        '\tat org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:339)\n'+
        '\tat org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:371)\n'+
        '\tat org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:337)\n'+
        '\tat org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:202)\n'+
        '\tat org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1681)\n'+
        '\tat org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1627)\n'+
        '\tat org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.resolveFieldValue(AutowiredAnnotationBeanPostProcessor.java:785)\n'+
        '\t... 20 common frames omitted\n'+
        'Caused by: com.baomidou.dynamic.datasource.exception.ErrorCreateDataSourceException: druid create error\n'+
        '\tat com.baomidou.dynamic.datasource.creator.druid.DruidDataSourceCreator.createDataSource(DruidDataSourceCreator.java:134)\n'+
        '\tat com.baomidou.dynamic.datasource.creator.DefaultDataSourceCreator.createDataSource(DefaultDataSourceCreator.java:97)\n'+
        '\tat com.baomidou.dynamic.datasource.provider.AbstractDataSourceProvider.createDataSourceMap(AbstractDataSourceProvider.java:53)\n'+
        '\tat com.baomidou.dynamic.datasource.provider.YmlDynamicDataSourceProvider.loadDataSources(YmlDynamicDataSourceProvider.java:53)\n'+
        '\tat com.baomidou.dynamic.datasource.DynamicRoutingDataSource.afterPropertiesSet(DynamicRoutingDataSource.java:225)\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.invokeInitMethods(AbstractAutowireCapableBeanFactory.java:1865)\n'+
        '\tat org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1814)\n'+
        '\t... 29 common frames omitted\n'+
        'Caused by: org.postgresql.util.PSQLException: Connection to 127.0.0.1:5432 refused. Check that the hostname and port are correct and that the postmaster is accepting TCP/IP connections.\n'+
        '\tat org.postgresql.core.v3.ConnectionFactoryImpl.openConnectionImpl(ConnectionFactoryImpl.java:303)\n'+
        '\tat org.postgresql.core.ConnectionFactory.openConnection(ConnectionFactory.java:51)\n'+
        '\tat org.postgresql.jdbc.PgConnection.<init>(PgConnection.java:225)\n'+
        '\tat org.postgresql.Driver.makeConnection(Driver.java:465)\n'+
        '\tat org.postgresql.Driver.connect(Driver.java:264)\n'+
        '\tat com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:132)\n'+
        '\tat com.alibaba.druid.filter.FilterAdapter.connection_connect(FilterAdapter.java:764)\n'+
        '\tat com.alibaba.druid.filter.FilterEventAdapter.connection_connect(FilterEventAdapter.java:33)\n'+
        '\tat com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:126)\n'+
        '\tat com.alibaba.druid.filter.stat.StatFilter.connection_connect(StatFilter.java:244)\n'+
        '\tat com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:126)\n'+
        '\tat com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1687)\n'+
        '\tat com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1803)\n'+
        '\tat com.alibaba.druid.pool.DruidDataSource.init(DruidDataSource.java:595)\n'+
        '\tat com.baomidou.dynamic.datasource.creator.druid.DruidDataSourceCreator.createDataSource(DruidDataSourceCreator.java:132)\n'+
        '\t... 35 common frames omitted\n'+
        'Caused by: java.net.ConnectException: Connection refused: getsockopt\n'+
        '\tat java.base/sun.nio.ch.Net.pollConnect(Native Method)\n'+
        '\tat java.base/sun.nio.ch.Net.pollConnectNow(Net.java:684)\n'+
        '\tat java.base/sun.nio.ch.NioSocketImpl.timedFinishConnect(NioSocketImpl.java:547)\n'+
        '\tat java.base/sun.nio.ch.NioSocketImpl.connect(NioSocketImpl.java:602)\n'+
        '\tat java.base/java.net.SocksSocketImpl.connect(SocksSocketImpl.java:327)\n'+
        '\tat java.base/java.net.Socket.connect(Socket.java:633)\n'+
        '\tat org.postgresql.core.PGStream.createSocket(PGStream.java:231)\n'+
        '\tat org.postgresql.core.PGStream.<init>(PGStream.java:95)\n'+
        '\tat org.postgresql.core.v3.ConnectionFactoryImpl.tryConnect(ConnectionFactoryImpl.java:98)\n'+
        '\tat org.postgresql.core.v3.ConnectionFactoryImpl.openConnectionImpl(ConnectionFactoryImpl.java:213)\n'+
        '\t... 49 common frames omitted\n'+
        '2026-09-18 15:28:57.209 [Druid-ConnectionPool-Create-1924874046] ERROR com.alibaba.druid.pool.DruidDataSource:2592 - create connection SQLException, url: jdbc:postgresql://127.0.0.1:5432/postgres, errorCode 0, state 08001\n'+
        'org.postgresql.util.PSQLException: Connection to 127.0.0.1:5432 refused. Check that the hostname and port are correct and that the postmaster is accepting TCP/IP connections.\n'+
        '\tat org.postgresql.core.v3.ConnectionFactoryImpl.openConnectionImpl(ConnectionFactoryImpl.java:303)\n'+
        '\tat org.postgresql.core.ConnectionFactory.openConnection(ConnectionFactory.java:51)\n'+
        '\tat org.postgresql.jdbc.PgConnection.<init>(PgConnection.java:225)\n'+
        '\tat org.postgresql.Driver.makeConnection(Driver.java:465)\n'+
        '\tat org.postgresql.Driver.connect(Driver.java:264)\n'+
        '\tat com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:132)\n'+
        '\tat com.alibaba.druid.filter.FilterAdapter.connection_connect(FilterAdapter.java:764)\n'+
        '\tat com.alibaba.druid.filter.FilterEventAdapter.connection_connect(FilterEventAdapter.java:33)\n'+
        '\tat com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:126)\n'+
        '\tat com.alibaba.druid.filter.stat.StatFilter.connection_connect(StatFilter.java:244)\n'+
        '\tat com.alibaba.druid.filter.FilterChainImpl.connection_connect(FilterChainImpl.java:126)\n'+
        '\tat com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1687)\n'+
        '\tat com.alibaba.druid.pool.DruidAbstractDataSource.createPhysicalConnection(DruidAbstractDataSource.java:1803)\n'+
        '\tat com.alibaba.druid.pool.DruidDataSource$CreateConnectionThread.run(DruidDataSource.java:2590)\n'+
        'Caused by: java.net.ConnectException: Connection refused: getsockopt\n'+
        '\tat java.base/sun.nio.ch.Net.pollConnect(Native Method)\n'+
        '\tat java.base/sun.nio.ch.Net.pollConnectNow(Net.java:684)\n'+
        '\tat java.base/sun.nio.ch.NioSocketImpl.timedFinishConnect(NioSocketImpl.java:547)\n'+
        '\tat java.base/sun.nio.ch.NioSocketImpl.connect(NioSocketImpl.java:602)\n'+
        '\tat java.base/java.net.SocksSocketImpl.connect(SocksSocketImpl.java:327)\n'+
        '\tat java.base/java.net.Socket.connect(Socket.java:633)\n'+
        '\tat org.postgresql.core.PGStream.createSocket(PGStream.java:231)\n'+
        '\tat org.postgresql.core.PGStream.<init>(PGStream.java:95)\n'+
        '\tat org.postgresql.core.v3.ConnectionFactoryImpl.tryConnect(ConnectionFactoryImpl.java:98)\n'+
        '\tat org.postgresql.core.v3.ConnectionFactoryImpl.openConnectionImpl(ConnectionFactoryImpl.java:213)\n'+
        '\t... 13 common frames omitted\n'+
        '\n'+
        'Process finished with exit code 1\n 2号店',
      total: 1274.56,
    },
    {
      name: '滨化狼人  1号店',
      total: 1234.56,
    },
    {
      name: '沐沐筱 2号店',
      total: 1274.56,
    },
    {
      name: '杰尼龟 1号店',
      total: 1234.56,
    },
  ]);

  const inOrderBarData = ref([
    {
      name: `1月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
    {
      name: `3月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
    {
      name: `4月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
    {
      name: `5月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
    {
      name: `6月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
    {
      name: `7月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
    {
      name: `8月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
    {
      name: `9月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
    {
      name: `10 月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
    {
      name: `11月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
    {
      name: `12月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
  ]);
  const outOrderBarData = ref([
    {
      name: `1月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
    {
      name: `3月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
    {
      name: `4月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
    {
      name: `5月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
    {
      name: `6月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
    {
      name: `7月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
    {
      name: `8月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
    {
      name: `9月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
    {
      name: `10 月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
    {
      name: `11月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
    {
      name: `12月`,
      value: Math.floor(Math.random() * 1000) + 200,
    },
  ]);
  const seriesColor = computed(() => {
    return getThemeColor.value;
  });

  setTimeout(() => {
    loading.value = false;
  }, 500);

  const indexBottomTab = ref('1');
  const indexRegisterType = ref('转移登记');
  const dataSource = ref([]);
  const ipagination = ref(table.ipagination);

  function tableChange(pagination) {
    ipagination.value.current = pagination.current;
    ipagination.value.pageSize = pagination.pageSize;
    loadDataSource();
  }

  function loadDataSource() {
    dataSource.value = table.dataSource;
  }

  loadDataSource();

  //加载待办任务
  todoTaskList();
  //加载入库趋势
  inOrderData();
  //加载出库趋势
  outOrderData();
  //加载货主出库统计
  rankListData();

  /**
   * 获取仓库信息
   */
  async function getTodoTaskList(params?) {
    return defHttp.get({ url: '/bigscreen/todo-tasks', params });
  }
  /**
   *加载待办任务
   */
  async function todoTaskList() {
    const result = await getTodoTaskList();
    console.log('records', result);
    if (result && result.length > 0) {
      growCardList.value = result;
    }
  }
  /**
   * 获取入库趋势
   */
  async function getInOrderData(params?) {
    return defHttp.get({ url: '/bigscreen/todo-tasks', params });
  }
  /**
   * 加载入库趋势
   */
  async function inOrderData() {
    const result = await getInOrderData();
    console.log('records', result);
    if (result && result.length > 0) {
      inOrderBarData.value = result;
    }
  }
  /**
   * 获取出库趋势
   */
  async function getOutOrderData(params?) {
    // return defHttp.get({ url: '/bigscreen/todo-tasks', params });
  }
  /**
   * 加载入库趋势
   */
  async function outOrderData() {
    const result = await getOutOrderData();
    console.log('records', result);
    if (result && result.length > 0) {
      outOrderBarData.value = result;
    }
  }
  /**
   * 获取货主出库统计
   */
  async function getRankList(params?) {
    // return defHttp.get({ url: '/bigscreen/todo-tasks', params });
  }
  /**
   * 加载货主出库统计
   */
  async function rankListData() {
    const result = await getRankList();
    console.log('records', result);
    if (result && result.length > 0) {
      rankList.value = result;
    }
  }

  // 暴露方法给父组件
  defineExpose({
    todoTaskList,
    inOrderData,
    outOrderData,
    rankListData,
  });
</script>

<style lang="less" scoped>
  .infoArea {
    display: flex;
    justify-content: space-between;
    padding: 0 10%;
    .head-info.center {
      padding: 0;
    }
    .head-info {
      min-width: 0;
    }
  }
  .circle-cust {
    position: relative;
    top: 28px;
    left: -100%;
  }

  .extra-wrapper {
    line-height: 55px;
    padding-right: 24px;

    .extra-item {
      display: inline-block;
      margin-right: 24px;

      a {
        margin-left: 24px;
      }
    }
  }

  /* 首页访问量统计 */
  .head-info {
    position: relative;
    text-align: left;
    padding: 0 32px 0 0;
    min-width: 125px;

    &.center {
      text-align: center;
      padding: 0 32px;
    }

    span {
      color: rgba(0, 0, 0, 0.45);
      display: inline-block;
      font-size: 0.95rem;
      line-height: 42px;
      margin-bottom: 4px;
    }

    p {
      line-height: 42px;
      margin: 0;

      a {
        font-weight: 600;
        font-size: 1rem;
      }
    }
  }
  .ant-card {
    ::v-deep(.ant-card-head-title) {
      font-weight: normal;
    }
  }
</style>
