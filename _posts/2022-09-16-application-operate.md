---
layout: post
title: 云原生应用管理
category: IaC
catalog: true
published: true
tags:
  - IaC
time: '2022.09.16 11:34:00'
---
# 一、开通([provisioning](https://www.redhat.com/en/topics/automation/what-is-provisioning#overview))
`provisioning`是设置IT基础设施的过程。`provisioning`和配置不是一个事情，但是他们都是`deployment`过程中的步骤。有多种类别的`provisioning`：`server provisioning`(服务器开通),`network provisioning`(网络开通),`user provisioning`(用户开通),`service provisioning`(服务开通)。  
`provisioning`和`IaC`是什么关系呢？我个人理解前者是目标，后者是实施方案之一。

# 二、特征(traits)
## 2.2 [oam中的traits](https://github.com/oam-dev/spec/blob/master/6.traits.md)
`open application model`中给出定义，`traits`的目的是想通过此配置对分布式应用进行控制配置，示例配置：
```
apiVersion: core.oam.dev/v1beta1
kind: Application
metadata:
  name: my-example-app
  annotations:
    version: v1.0.0
    description: "Brief description of the application"
spec:
  components:
    - name: publicweb
      type: web-ui
      properties: # properties targeting component parameters.
        image: example/web-ui:v1.0.2@sha256:verytrustworthyhash
        param_1: "enabled" # param_1 is defined on the web-ui component
      traits:
        - type: ingress # ingress trait providing a public endpoint for the publicweb component of the application.
          properties: # properties are defined by the trait CRD spec. This example assumes path and port.
            path: /
            port: 8080
      scopes:
        "healthscopes.core.oam.dev": "app-health" # An application level health scope including both components.
    - name: backend
      type: company/test-backend # test-backend is referenced from other namespace
      properties:
        debug: "true" # debug is a parameter defined in the test-backend component.
      traits:
        - type: scaler # scaler trait to specify the number of replicas for the backend component
          properties:
            replicas: 4
      scopes:
        "healthscopes.core.oam.dev": "app-health" # An application level health scope including both components.
```
traits对象定义如下所示。
```
  trait:
    type: object
    description: The trait section defines traits that will be used in a component
      instance.
    properties:
      name:
        type: string
        description: The name of the trait instance. Must be unique to the deployment
          environment.
      properties:
        type: object
        description: Overrides of parameters that are exposed by the trait type defined
          in 'type'.
        additionalProperties:
          "$ref": "#/definitions/propertiesObject"
    required:
    - name
    additionalProperties: false
  propertiesObject:
    type: object
    description: A properties object (for trait and scope configuration) is an object
      whose structure is determined by the trait or scope property schema. It may
      be a simple value, or it may be a complex object.
    properties: {}
    additionalProperties: true
```

## 2.2 [os-traits](https://github.com/openstack/os-traits/blob/master/os_traits/compute/arch.py)
openstack对traits也有类似定义，实际对[openstack trait描述](https://specs.openstack.org/openstack/nova-specs/specs/pike/implemented/resource-provider-traits.html)和OAM中的trait中的定义极其相似，是对云资源属性特征的提炼和应用。

# 三、参考文献
1. [kubevela trait](https://github.com/kubevela/kubevela/tree/master/references/docgen/def-doc/trait)
2. [kubevela ingress.cue](https://github.com/kubevela/kubevela/blob/master/vela-templates/definitions/deprecated/ingress.cue)
3. [kubevela ingress.yaml](https://github.com/kubevela/kubevela/blob/master/charts/vela-core/templates/defwithtemplate/ingress.yaml)  
4. [provisioning](https://zh.wikipedia.org/zh-tw/%E6%9C%8D%E5%8A%A1%E5%BC%80%E9%80%9A)
5. [kubevela webservice.cue](https://github.com/kubevela/kubevela/blob/master/vela-templates/definitions/internal/component/webservice.cue)