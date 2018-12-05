# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

swagger: '2.0'
info:
  title: Service Center API
  version: "4.0.0"
# the domain of the service
host: 127.0.0.1:30100
# array of all schemes that your API supports
schemes:
  - http
  - https
# will be prefixed to all paths
produces:
  - application/json
paths:
  /v4/{project}/registry/version:
    get:
      description: |
        查询服务中心版本信息。
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
      tags:
        - base
      responses:
        200:
          description: 版本信息结构体
          schema:
            $ref: '#/definitions/Version'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/health:
    get:
      description: |
        查询服务中心集群信息。
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
      tags:
        - base
      responses:
        200:
          description: 服务中心实例集群信息列表
          schema:
            $ref: '#/definitions/GetInstancesResponse'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/microservices/{serviceId}:
    get:
      description: |
        根据serviceId查询微服务定义信息。
      operationId: getOne
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务唯一标识。
          required: true
          type: string
      tags:
        - microservices
      responses:
        200:
          description: 微服务结构体
          schema:
            $ref: '#/definitions/GetMicroServicesResponse'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
    delete:
      description: |
        删除一个微服务定义及其相关信息，同时注销其所有实例信息。
      operationId: delete
      parameters:
        - name: x-domain-name
          in: header
          required: true
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务唯一标识。
          required: true
          type: string
        - name: force
          in: query
          description: 不传即默认为false。true表示强制删除，则与该服务相关的信息删除，false表示非强制删除：如果作为该被依赖（作为provider，提供服务,且不是只存在自依赖）或者存在实例，则不能删除,其它均删除。
          type: boolean
      tags:
        - microservices
      responses:
        200:
          description: 修改成功
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/microservices:
    get:
      description: |
        根据条件组合，查询满足所有条件的微服务定义信息。
      operationId: getServices
      tags:
        - microservices
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
      responses:
        200:
          description: 查询成功
          schema:
            $ref: '#/definitions/GetMicroServicesResponse'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
    post:
      description: |
        在注册微服务实例前需要创建服务静态信息，之后注册的微服务实例根据service id这个字段与静态信息关联，一个服务对应对多个实例。
      operationId: create
      parameters:
        - name: x-domain-name
          in: header
          required: true
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: service
          in: body
          description: 创建微服务请求结构体。
          required: true
          schema:
            $ref: '#/definitions/CreateMicroService'
      tags:
        - microservices
      responses:
        200:
          description: 创建成功
          schema:
            $ref: '#/definitions/CreateMicroServiceResponse'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
    delete:
      description: |
        批量删除指定的微服务定义及其相关信息，同时注销其所有实例信息。
      operationId: deleteServices
      parameters:
        - name: project
          in: path
          required: true
          type: string
        - name: serviceIds
          in: body
          description: 批量删除服务的服务ID列表
          required: true
          schema:
            $ref: '#/definitions/DelServicesRequest'
      tags:
        - microservices
      responses:
        200:
          description: 删除成功
          schema:
            $ref: '#/definitions/DelServicesResponse'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/microservices/{serviceId}/properties:
    put:
      description: |
        创建微服务静态信息后可对服务部分字段进行更新，每次更新都需要传入完整的服务静态信息json，也就是说，即便不更新部分的字段也要作为json的属性传过去。
      operationId: updateProperties
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务唯一标识。
          required: true
          type: string
        - name: properties
          in: body
          description: 微服务扩展属性请求结构体。
          required: true
          schema:
            $ref: '#/definitions/UpdateProperties'
      tags:
        - microservices
      responses:
        200:
          description: 修改成功
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/microservices/{serviceId}/tags:
    post:
      description: |
        为serviceId的微服务创建tag。
      operationId: addTags
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务唯一标识。
          required: true
          type: string
        - name: tags
          in: body
          description: 要创建的tags。
          required: true
          schema:
            $ref: '#/definitions/Tags'
      tags:
        - microservices
        - tags
      responses:
        200:
          description: 创建成功
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
    get:
      description: |
        获取serviceId的微服务的tag
      operationId: getTags
      parameters:
        - name: x-domain-name
          in: header
          required: true
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务唯一标识。
          required: true
          type: string
      tags:
        - microservices
        - tags
      responses:
        200:
          description: 查询成功
          schema:
            $ref: '#/definitions/Tags'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/microservices/{serviceId}/tags/{key}:
    put:
      description: |
        为serviceId的微服务更新key对应的value值
      operationId: updateTag
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务唯一标识。
          required: true
          type: string
        - name: key
          in: path
          description: 要更新的tag的key值。
          required: true
          type: string
        - name: value
          in: query
          description: 要更新的tag的value值。
          required: true
          type: string
      tags:
        - microservices
        - tags
      responses:
        200:
          description: 更新成功
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
    delete:
      description: |
        为serviceId的微服务删除tags
      operationId: deleteTags
      parameters:
        - name: x-domain-name
          in: header
          required: true
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务唯一标识。
          required: true
          type: string
        - name: key
          in: path
          description: 要删除的tag的key值，多个key的话，以，隔开。
          required: true
          type: string
      tags:
        - microservices
        - tags
      responses:
        200:
          description: 删除成功
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/microservices/{serviceId}/rules:
    post:
      description: |
        为serviceId的服务新增黑白名单，同一服务，attribute和pattern唯一标识一份黑白名单。
      operationId: addRule
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务唯一标识。
          required: true
          type: string
        - name: rules
          in: body
          description: 新增黑白名单。
          required: true
          schema:
            $ref: '#/definitions/AddRules'
      tags:
        - microservices
        - rules
      responses:
        200:
          description: 创建成功
          schema:
            $ref: '#/definitions/AddRuleResponse'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
    get:
      description: |
        获取serviceId的服务的黑白名单信息。
      operationId: getRule
      parameters:
        - name: x-domain-name
          in: header
          required: true
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务唯一标识。
          required: true
          type: string
      tags:
        - microservices
        - rules
      responses:
        200:
          description: 查询成功
          schema:
            $ref: '#/definitions/Rules'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/microservices/{serviceId}/rules/{rule_id}:
    put:
      description: |
        为serviceId的服务更新黑白名单。
      operationId: updateRule
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务唯一标识。
          required: true
          type: string
        - name: rule_id
          in: path
          description: ruleId。
          required: true
          type: string
        - name: rule
          in: body
          description: 要更新的rule
          required: true
          schema:
            $ref: "#/definitions/AddOrUpdateRule"
      tags:
        - microservices
        - rules
      responses:
        200:
          description: 修改成功
          schema:
            type: string
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
    delete:
      description: |
        为serviceId的服务删除黑白名单。
      operationId: deleteRule
      parameters:
        - name: x-domain-name
          in: header
          required: true
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务唯一标识。
          required: true
          type: string
        - name: rule_id
          in: path
          description: ruleId。
          required: true
          type: string
      tags:
        - microservices
        - rules
      responses:
        200:
          description: 删除成功
          schema:
            type: string
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/microservices/{serviceId}/schemas/{schemaId}:
    get:
      description: |
        根据serviceId和schemaId查询微服务的schema信息。
      operationId: getSchemaInfo
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务唯一标识。
          required: true
          type: string
        - name: schemaId
          in: path
          description: 微服务契约唯一标识。
          required: true
          type: string
      tags:
        - microservices
        - schemas
      responses:
        200:
          description: 查询成功,如果summary存在，则header里面的X-Schema-Summary的value为该schema对应的摘要
          headers:
            X-Schema-Summary:
              type: string
          schema:
            $ref: '#/definitions/getSchemaInfoResponse'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
    put:
      description: |
        根据schemaId更新微服务的访问契约内容。
      operationId: modifySchema
      parameters:
        - name: x-domain-name
          in: header
          required: true
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务唯一标识。
          required: true
          type: string
        - name: schemaId
          in: path
          description: 微服务契约唯一标识。
          required: true
          type: string
        - name: schema
          in: body
          description: 微服务契约内容。
          required: true
          schema:
            $ref: '#/definitions/CreateSchema'
      tags:
        - microservices
        - schemas
      responses:
        200:
          description: 修改成功
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
    delete:
      description: |
        删除微服务的一个schema信息。
      operationId: deleteSchema
      parameters:
        - name: x-domain-name
          in: header
          required: true
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务唯一标识。
          required: true
          type: string
        - name: schemaId
          in: path
          description: 微服务契约唯一标识。
          required: true
          type: string
      tags:
        - microservices
        - schemas
      responses:
        200:
          description: 删除成功
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/microservices/{serviceId}/schemas:
    post:
      description: |
        批量上传schemas。
      operationId: ModifySchemas
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 唯一标识。
          required: true
          type: string
        - name: type
          in: body
          required: true
          description: 批量上传schemas信息。
          schema:
            $ref: '#/definitions/ModifySchemasRequest'
      tags:
        - microservices
        - schemas
      responses:
        200:
          description: 創建成功
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
    get:
      description: |
        批量查询所有schemas和summary。
      operationId: GetAllSchemas
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 唯一标识。
          required: true
          type: string
        - name: withSchema
          in: query
          description: 是否查询schema，0只显示summary，1同时显示schema。
          type: integer
          default: 0
      tags:
        - microservices
        - schemas
      responses:
        200:
          description: 查询成功
          schema:
            $ref: '#/definitions/AllSchemasRequest'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/dependencies:
    post:
      description: |
        增量式添加服务间的依赖关系，consumer的version必须是确定的版本，serviceName不能为*，consumer必须是已存在的服务，provider可以是还未创建的。
      operationId: AddDependenciesForMicroServices
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: type
          in: body
          description: 创建服务间的依赖关系的请求体。
          required: true
          schema:
            $ref: '#/definitions/CreateDependenciesRequest'
      tags:
        - dependencies
      responses:
        200:
          description: 创建成功
          schema:
            type: string
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
    put:
      description: |
        覆盖式修改服务间的依赖关系，consumer的version必须是确定的版本，serviceName不能为*，consumer必须是已存在的服务，provider可以是还未创建的。
      operationId: createDependenciesForMircServices
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: type
          in: body
          description: 创建服务间的依赖关系的请求体。
          required: true
          schema:
            $ref: '#/definitions/CreateDependenciesRequest'
      tags:
        - dependencies
      responses:
        200:
          description: 创建成功
          schema:
            type: string
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/microservices/{consumerId}/providers:
    get:
      description: |
        根据consumerId获取该服务的所有providers
      operationId: getConsumerDependencies
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: consumerId
          in: path
          description: 消费者的服务id。
          required: true
          type: string
        - name: noSelf
          in: query
          description: 是否取消返回自依赖的关系
          type: integer
          default: 0
        - name: sameDomain
          in: query
          description: 是否取消返回共享服务的关系
          type: integer
          default: 0
      tags:
        - dependencies
      responses:
        200:
          description: 查询成功
          schema:
            $ref: '#/definitions/GetConDependenciesResponse'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/microservices/{providerId}/consumers:
    get:
      description: |
        根据providerId获取该服务的所有consumers
      operationId: getProviderDependencies
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: providerId
          in: path
          description: 提供者的服务id。
          required: true
          type: string
        - name: noSelf
          in: query
          description: 是否取消返回自依赖的关系
          type: integer
          default: 0
        - name: sameDomain
          in: query
          description: 是否取消返回共享服务的关系
          type: integer
          default: 0
      tags:
        - dependencies
      responses:
        200:
          description: 查询成功
          schema:
            $ref: '#/definitions/GetProDependenciesResponse'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/existence:
    get:
      description: |
        可通过指定条件，查询微服务serviceId或schema的唯一标识信息。
      operationId: exist
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: type
          in: query
          description: 资源类型 microservice微服务 schema微服务访问契约。
          required: true
          type: string
        - name: env
          in: query
          description: development|testing|acceptance|production
          type: string
          default: development
        - name: appId
          in: query
          description: 资源类型为 microservice时 需传入应用app唯一标识。
          type: string
          required: true
        - name: serviceName
          in: query
          description: 资源类型为 microservice时 需传入微服务名称。
          type: string
          required: true
        - name: version
          in: query
          description: 资源类型为 microservice时 需传入微服务版本。
          type: string
          required: true
        - name: serviceId
          in: query
          description: 资源类型为 schema时 需传入微服务唯一标识。
          type: string
          required: true
        - name: schemaId
          in: query
          description: 资源类型为 schema时 需传入schema唯一标识。
          type: string
          required: true
      tags:
        - schemas
        - microservices
      responses:
        200:
          description: 查询成功,查询shema存不存时，如果summary存在，则在header里面返回，key为X-Schema-Summary
          headers:
            X-Schema-Summary:
              type: string
          schema:
            $ref: '#/definitions/GetResourceResponse'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/microservices/{serviceId}/instances:
    post:
      description: |
        创建微服务后就可以注册该微服务的实例了。 注册微服务实例时，需提供该微服务实例相关的信息。
        instanceID可定制,如果定制了，再次注册就直接全内容覆盖。如果没定制，逻辑如下：系统自动生成id，如果endpoints内容重复，则使用原来的id
      operationId: register
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务唯一标识。
          required: true
          type: string
        - name: instance
          in: body
          description: 微服务实例请求结构体。
          required: true
          schema:
            $ref: '#/definitions/CreateInstance'
      tags:
        - instances
      responses:
        200:
          description: 注册成功
          schema:
            $ref: '#/definitions/CreateInstanceResponse'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
    get:
      description: |
        实例注册后可以根据 service_id 发现该微服务的所有实例。
      operationId: getInstances
      parameters:
        - name: x-domain-name
          in: header
          required: true
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: X-ConsumerId
          in: header
          description: 微服务消费者的微服务唯一标识。
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务唯一标识。
          required: true
          type: string
        - name: tags
          in: query
          description: Tag标签过滤，多个时逗号分隔。
          type: string
        - name: env
          in: query
          description: 实例的environment。
          type: string
      tags:
        - instances
      responses:
        200:
          description: 查询成功
          schema:
            $ref: '#/definitions/GetInstancesResponse'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/microservices/{serviceId}/instances/{instanceId}:
    delete:
      description: |
        实例注册后可以根据 instance_id 进行实例注销。
      operationId: unregister
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务唯一标识。
          required: true
          type: string
        - name: instanceId
          in: path
          description: 微服务实例唯一标识。
          required: true
          type: string
      tags:
        - instances
      responses:
        200:
          description: 注销成功
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
    get:
      description: |
        实例注册后可以根据 service_id和serviceId获取该实例的详细信息。
      operationId: GetOneInstance
      parameters:
        - name: x-domain-name
          in: header
          required: true
          type: string
          default: default
        - name: X-ConsumerId
          in: header
          description: 微服务消费者的微服务唯一标识。
          required: true
          type: string
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务唯一标识。
          required: true
          type: string
        - name: instanceId
          in: path
          description: 实例唯一标识。
          required: true
          type: string
        - name: tags
          in: query
          description: Tag标签过滤，多个时逗号分隔。
          type: string
        - name: env
          in: query
          description: 实例的environment。
          type: string
      tags:
        - instances
      responses:
        200:
          description: 查询成功
          schema:
            $ref: '#/definitions/GetOneInstanceResponse'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/microservices/{serviceId}/instances/{instanceId}/properties:
    put:
      description: |
        实例注册后可以根据 instance_id 进行添加/更新一个微服务实例扩展信息。
      operationId: updateInstanceProperties
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务唯一标识。
          required: true
          type: string
        - name: instanceId
          in: path
          description: 微服务实例唯一标识。
          required: true
          type: string
        - name: properties
          in: body
          description: 微服务实例扩展属性请求结构体。
          required: true
          schema:
            $ref: '#/definitions/UpdateProperties'
      tags:
        - instances
      responses:
        200:
          description: 修改成功
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/microservices/{serviceId}/instances/{instanceId}/status:
    put:
      description: |
        实例注册后可以根据 instance_id 进行更新一个微服务实例状态。
      operationId: updateStatus
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务唯一标识。
          required: true
          type: string
        - name: instanceId
          in: path
          description: 微服务实例唯一标识。
          required: true
          type: string
        - name: value
          in: query
          description: 实例状态 UP在线OUTOFSERVICE摘机STARTING正在启动DOWN下线。
          required: true
          type: string
      tags:
        - instances
      responses:
        200:
          description: 修改成功
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/microservices/{serviceId}/instances/{instanceId}/heartbeat:
    put:
      description: |
        服务提供端需要向服务中心发送心跳信息，以保证服务中心知道服务实例是否健康。
      operationId: heartbeat
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务唯一标识。
          required: true
          type: string
        - name: instanceId
          in: path
          description: 微服务实例唯一标识。
          required: true
          type: string
      tags:
        - instances
      responses:
        200:
          description: 更新成功
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/heartbeats:
    put:
      description: |
        服务提供端需要向服务中心发送心跳信息，以保证服务中心知道服务实例是否健康。该接口为批量接口
      operationId: HeartbeatSet
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: Instances
          in: body
          description: 批量上报心跳的实例的标识。
          required: true
          schema:
            $ref: '#/definitions/HeartbeatSetRequest'
      tags:
        - instances
      responses:
        200:
          description: 更新成功
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/InstancesHbRst'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/instances:
    get:
      description: |
        实例注册后可以根据微服务版本规则或字段条件 发现该微服务的实例。同时会将此微服务版本规则记录到依赖关系中，当发现相同微服务的不同版本规则时，会覆盖已有的依赖关系。
      operationId: find
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: x-resource-revsion
          in: header
          type: string
          description: 客户端缓存的版本号,由上一次请求该API返回Header中获得;如请求版本号不为空且与服务端不匹配则服务端返回其最新的实例集合和版本号;如匹配则服务端返回304状态且Body为空。
        - name: X-ConsumerId
          in: header
          description: 微服务消费者的微服务唯一标识。
          type: string
        - name: project
          in: path
          required: true
          type: string
        - name: appId
          in: query
          description: 应用app唯一标识。
          required: true
          type: string
        - name: serviceName
          in: query
          description: 微服务名称。
          required: true
          type: string
        - name: version
          in: query
          description: 版本规则：1.精确版本匹配 2.后续版本匹配 3.最新版本 4.版本范围
          type: string
          required: true
        - name: tags
          in: query
          description: Tag标签过滤，多个时逗号分隔。
          type: string
        - name: env
          in: query
          description: 实例的environment。
          type: string
      tags:
        - instances
      responses:
        200:
          description: 查询成功
          headers:
            "X-Resource-Revision":
              type: "string"
              description: 返回集合的版本号,当集合内容发生变化,版本号随之变化
          schema:
            $ref: '#/definitions/GetInstancesResponse'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
    post:
      description: |
        批量微服务实例发现接口
      operationId: batchFind
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: X-ConsumerId
          in: header
          description: 微服务消费者的微服务唯一标识。
          type: string
        - name: project
          in: path
          required: true
          type: string
        - name: services
          in: body
          description: 查询微服务的请求结构体
          required: true
          schema:
            $ref: '#/definitions/BatchFindRequest'
      tags:
        - instances
      responses:
        200:
          description: 查询成功
          schema:
            $ref: '#/definitions/BatchFindResponse'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/microservices/{serviceId}/watcher:
    get:
      description: |
        当服务在心跳消失，注册，注销，状态更新时， 将这些变化主动推送到客户端。
      operationId: watch
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务消费者的微服务唯一标识。
          required: true
          type: string
      tags:
        - microservices
      responses:
        200:
          description: 实例变化时，成功推送给watcher的信息
          schema:
            $ref: '#/definitions/WatchInstanceResponse'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/registry/microservices/{serviceId}/listwatcher:
    get:
      description: |
        watch成功后返回完整的微服务提供者的实例信息，且服务在心跳消失，注册，注销，状态更新时，将这些变化主动推送到客户端。
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务消费者的微服务唯一标识。
          required: true
          type: string
      tags:
        - microservices
      responses:
        200:
          description: 推送给watcher实例变化信息
          schema:
            $ref: '#/definitions/WatchInstanceResponse'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/govern/microservices/{serviceId}:
    get:
      description: |
        查询单个服务的所有信息。
      operationId: GetServiceDetail
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
          required: true
        - name: project
          in: path
          required: true
          type: string
        - name: serviceId
          in: path
          description: 微服务消费者的微服务唯一标识。
          required: true
          type: string
      tags:
        - governance
      responses:
        200:
          description: 单个服务的信息
          schema:
            $ref: '#/definitions/GetServiceDetailResponse'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/govern/microservices:
    get:
      description: |
        查询单个服务的所有信息。
      operationId: GetServicesInfo
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
          required: true
        - name: project
          in: path
          required: true
          type: string
        - name: options
          in: query
          description: 获取对应options相对应的信息，all,tag,rules,instances,schemas,dependencies,statistics,没有默认返回服务信息
          required: false
          type: string
      tags:
        - governance
      responses:
        200:
          description: 单个服务的信息
          schema:
            $ref: '#/definitions/GetServicesInfoResponse'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/govern/relations:
    get:
      description: |
        查询服务间的关系。
      operationId: GetGraph
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
          description: 租户名字
          required: true
        - name: project
          in: path
          description: 项目名字
          required: true
          type: string
      tags:
        - governance
      responses:
        200:
          description: 单个服务的信息
          schema:
            $ref: '#/definitions/Graph'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/govern/apps:
    get:
      description: |
        查询所有appId。
      operationId: GetApplications
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
          description: 租户名字
          required: true
        - name: project
          in: path
          description: 项目名字
          required: true
          type: string
        - name: env
          in: query
          description: development|testing|acceptance|production
          type: string
      tags:
        - governance
      responses:
        200:
          description: appId集合
          schema:
            $ref: '#/definitions/GetAppsResponse'
        400:
          description: 错误的请求
          schema:
            $ref: '#/definitions/Error'
        500:
          description: 内部错误
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/admin/dump:
    get:
      description: |
        Dump the information of service center runtime
      operationId: dump
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
          description: default租户
          required: true
        - name: project
          in: path
          default: default
          description: default项目
          required: true
          type: string
      tags:
        - admin
      responses:
        200:
          description: dump information
          schema:
            $ref: '#/definitions/DumpResponse'
        403:
          description: Forbidden
          schema:
            $ref: '#/definitions/Error'
  /v4/{project}/admin/clusters:
    get:
      description: |
        Return the registry clusters managed by Service Center
      operationId: clusters
      parameters:
        - name: x-domain-name
          in: header
          type: string
          default: default
          description: default租户
          required: true
        - name: project
          in: path
          default: default
          description: default项目
          required: true
          type: string
      tags:
        - admin
      responses:
        200:
          description: clusters information
          schema:
            $ref: '#/definitions/ClustersResponse'
definitions:
  Version:
    type: object
    properties:
      version:
        type: string
      apiVersion:
        type: string
      buildTag:
        type: string
      goVersion:
        type: string
      os:
        type: string
      arch:
        type: string
      config:
        $ref: '#/definitions/Config'
  Properties:
    type: object
    description: 扩展属性
    additionalProperties:
      type: string
  Framework:
    type: object
    properties:
      name:
        type: string
        description: 微服务开发框架，默认值为UNKNOWN
      version:
        type: string
        description: 微服务开发框架版本号
  UpdateProperties:
    type: object
    properties:
      properties:
        $ref: '#/definitions/Properties'
  CreateSchema:
    type: object
    required:
      - schema
    properties:
      schema:
        type: string
      summary:
        type: string
        description: 新加入参数，后面创建schema，请尽量提供，shema的摘要
  GetResourceResponse:
    type: object
    properties:
      serviceId:
        type: string
      schemaId:
        type: string
  CreateMicroServiceResponse:
    type: object
    properties:
      serviceId:
        type: string
  CreateMicroService:
    type: object
    properties:
      service:
        $ref: '#/definitions/MicroService'
      rules:
        type: array
        items:
           $ref: "#/definitions/Rule"
      instances:
        type: array
        items:
           $ref: '#/definitions/MicroServiceInstance'
      tags:
           $ref: "#/definitions/Properties"

  GetMicroServicesResponse:
    type: object
    properties:
      services:
        type: array
        items:
          $ref: '#/definitions/MicroService'
  CreateInstance:
    type: object
    properties:
      instance:
        $ref: '#/definitions/MicroServiceInstance'
  CreateInstanceResponse:
    type: object
    properties:
      instanceId:
        type: string
  GetInstancesResponse:
    type: object
    properties:
      instances:
        type: array
        items:
          $ref: '#/definitions/MicroServiceInstance'
  GetOneInstanceResponse:
    type: object
    properties:
      instance:
        $ref: '#/definitions/MicroServiceInstance'
  WatchMicroServiceKey:
    type: object
    properties:
      appId:
        type: string
      serviceName:
        type: string
      version:
        type: string
  WatchInstanceResponse:
    type: object
    properties:
      action:
        type: string
        description: 分别有CREATE UPDATE DELETE三种事件
      key:
        $ref: '#/definitions/WatchMicroServiceKey'
      instance:
        $ref: '#/definitions/MicroServiceInstance'
  MicroService:
    type: object
    required:
    - serviceName
    properties:
      serviceId:
        type: string
      environment:
        type: string
        description: development|testing|acceptance|production
      appId:
        type: string
        description: 应用App唯一标识
      serviceName:
        type: string
        description: 微服务名称，同一个App要保证唯一，允许使用英文字母和数字
      version:
        type: string
        description: 微服务版本号
      description:
        type: string
        description: 微服务描述信息
      level:
        type: string
        description: 微服务层级，FRONT/MIDDLE/BACK
      registerBy:
        type: string
        description: 微服务注册方式，SDK/PLATFORM/SIDECAR/UNKNOWN
      schemas:
        type: array
        description: 微服务访问契约内容的外键ID
        items:
          type: string
      status:
        type: string
        description: 微服务状态，UP表示上线 DOWN表示下线，默认值UP
        enum:
        - UP
        - DOWN
      timestamp:
        type: string
        description: post 或者 put 不带该参数，timestamp是内部生成的，只有get 接口才返回该值
      modTimestamp:
        type: string
        description: 更新时间
      framework:
        $ref: '#/definitions/Framework'
      paths:
        type: array
        description: 服务路由
        items:
          $ref: '#/definitions/ServicePath'
      properties:
        $ref: '#/definitions/Properties'
  HealthCheck:
    type: object
    required:
    - mode
    - interval
    - times
    properties:
      mode:
        type: string
        description: check模式 push/pull
      port:
        type: integer
        description: 端口
      interval:
        type: integer
        description: check interval (second)
      times:
        type: integer
        description: retry times
  MicroServiceInstance:
    type: object
    required:
    - hostName
    - endpoints
    properties:
      instanceId:
        type: string
        description: 实例id,唯一标识。创建实例，instanceId由service-center产生
      serviceId:
        type: string
        description: 微服务唯一标识，创建实例时，以url里面的为准，不用这里的serviceId。
      version:
        type: string
        description: 微服务版本号
      hostName:
        type: string
      endpoints:
        type: array
        items:
          type: string
          description: 例:rest:127.0.0.1:8080
      status:
        type: string
        description: 实例状态，UP|DOWN|STARTING|OUTOFSERVICE，默认值UP
      properties:
        $ref: '#/definitions/Properties'
      healthCheck:
        $ref: '#/definitions/HealthCheck'
      dataCenterInfo:
        $ref: '#/definitions/DataCenterInfo'
      timestamp:
        type: string
        description: 实例创建时间戳，自动生成
      modTimestamp:
        type: string
        description: 更新时间
  FindService:
    type: object
    properties:
      service:
        $ref: '#/definitions/DependencyKey'
      rev:
        type: string
        description: 客户端缓存的版本号。
  BatchFindRequest:
    type: object
    properties:
      services:
        type: array
        items:
          $ref: '#/definitions/FindService'
  FindResult:
    type: object
    properties:
      index:
        type: integer
        description: 与请求数组对应的索引。
      rev:
        type: string
        description: 服务端返回集合版本,如跟客户端缓存版本号一致,则instances为空。
      instances:
        type: array
        items:
          $ref: '#/definitions/MicroServiceInstance'
  FindFailedResult:
    type: object
    properties:
      indexes:
        type: array
        items:
          type: integer
        description: 与请求数组对应的索引集合。
      error:
        $ref: '#/definitions/Error'
  BatchFindResponse:
    type: object
    properties:
      failed:
        type: array
        items:
          $ref: '#/definitions/FindFailedResult'
      notModified:
        type: array
        items:
          type: integer
        description: 与请求数组对应的索引集合。
      updated:
        type: array
        items:
          $ref: '#/definitions/FindResult'
  CreateDependenciesRequest:
    type: object
    properties:
      dependencies:
        type: array
        items:
          $ref: '#/definitions/MicroServiceDependency'
  MicroServiceDependency:
    type: object
    properties:
      consumer:
        $ref: '#/definitions/DependencyKey'
      providers:
        type: array
        items:
          $ref: '#/definitions/DependencyKey'
  DependencyKey:
    type: object
    required:
        - appId
        - serviceName
        - version
    properties:
      environment:
        type: string
        description: development|testing|acceptance|production
      appId:
        type: string
        description: 应用app唯一标识。
      serviceName:
        type: string
        description: 微服务名称，作为provider支持为*，表示依赖同一租户下的所有服务,当服务名称为*的时候，appId和version可以省略，consumer不支持*。
      version:
        type: string
        description: 微服务版本，作为provider支持+，如1.0.1+[表示1.0.1以上的版本(包括1.0.1)]、固定版本和latest(当前最新版本)，作为consumer只能为固定版本。

  GetProDependenciesResponse:
    type: object
    properties:
      dependency:
        type: array
        items:
          $ref: "#/definitions/ProDependency"
  ProDependency:
    type: object
    properties:
      Consumers:
          $ref: "#/definitions/MicroService"
  GetConDependenciesResponse:
    type: object
    properties:
      dependency:
        type: array
        items:
          $ref: "#/definitions/ConDependency"
  ConDependency:
    type: object
    properties:
      Providers:
        $ref: "#/definitions/MicroService"
  Tags:
    type: object
    properties:
      tags:
        $ref: "#/definitions/Properties"

  Rules:
    type: object
    properties:
      rules:
        type: array
        items:
          $ref: "#/definitions/Rule"
  Rule:
    type: object
    properties:
      ruleId:
        description:  自定义ruleId
        type: string
      ruleType:
        description:  rule类型，WHITE或者BLACK
        type: string
      attribute:
        description:  如果是tag_xxx开头，则按Tag过滤attribute属性，否则，则按"ServiceId", "AppId", "ServiceName", "Version", "Description", "Level", "Status"过滤
        type: string
      pattern:
        description:  匹配规则，正则表达式，长度1到64
        type: string
      description:
        description:  rule描述
        type: string
      timestamp:
        description:  只有获取rule时返回使用，创建rule的时间
        type: string
      modTimestamp:
        type: string
        description: 更新时间
  AddRules:
    type: object
    properties:
      rules:
        type: array
        items:
          $ref: "#/definitions/AddOrUpdateRule"
  AddOrUpdateRule:
    type: object
    properties:
      ruleType:
        description:  rule类型，WHITE或者BLACK
        type: string
      attribute:
        description:  如果是tag_xxx开头，则按Tag过滤attribute属性，否则，则按"ServiceId", "AppId", "ServiceName", "Version", "Description", "Level", "Status"过滤
        type: string
      pattern:
        description:  匹配规则，正则表达式，长度1到64
        type: string
      description:
        description:  rule描述
        type: string
  DataCenterInfo:
    type: object
    required:
    - name
    - region
    - availableZone
    properties:
      name:
        type: string
        description: 区域名字
      region:
        type: string
        description: 区域
      availableZone:
        type: string
        description: 可获取区
  ServicePath:
    type: object
    properties:
      Path:
        type: string
        description: 路由地址
      Property:
          $ref: '#/definitions/Properties'
  AddRuleResponse:
    type: object
    properties:
      RuleIds:
        description: 生成的ruleId集合
        type: array
        items:
          type: string
  HeartbeatSetRequest:
    type: object
    properties:
      Instances:
        type: array
        items:
          $ref: "#/definitions/HeartbeatSetElement"
  HeartbeatSetElement:
    type: object
    properties:
      serviceId:
        description: 微服务id
        type: string
      instanceId:
        description: 微服务实例id
        type: string
  InstancesHbRst:
    type: object
    properties:
      instances:
        type: array
        items:
          $ref: "#/definitions/InstanceHbRst"
  InstanceHbRst:
    type: object
    properties:
      serviceId:
        description: 微服务id
        type: string
      instanceId:
        description: 微服务实例id
        type: string
      errMessage:
        description: 错误信息，成功为空，不成功，则为错误，在部分成功的场景使用
        type: string

  DelServicesRequest:
    type: object
    properties:
      serviceIds:
        type: array
        items:
          type: string
      force:
        description: 不传即默认为false。 强制删除，则与该服务相关的信息删除，非强制删除： 如果作为该被依赖（作为provider，提供服务,且不是只存在自依赖）或者存在实例，则不能删除,其它均删除。
        type: boolean

  DelServicesResponse:
    type: object
    properties:
      services:
        type: array
        items:
          $ref: "#/definitions/DelServicesRspInfo"

  DelServicesRspInfo:
    type: object
    properties:
      serviceId:
        description: 微服务id
        type: string
      errMessage:
        description: 错误信息，成功为空，不成功，则为错误，在部分成功的场景使用
        type: string
  ModifySchemasRequest:
     type: object
     properties:
       schemas:
         type: array
         items:
           $ref: "#/definitions/Schema"
  AllSchemasRequest:
     type: object
     properties:
       schemas:
         type: array
         items:
           $ref: "#/definitions/Schema"
  Schema:
     type: object
     properties:
       schemaId:
         type: string
       schema:
         type: string
       summary:
         type: string
  GetServiceDetailResponse:
     type: object
     properties:
        service:
         $ref: "#/definitions/ServiceDetail"
  ServiceDetail:
     type: object
     properties:
       microservice:
         $ref: "#/definitions/MicroService"
       instances:
         type: array
         description: 该服务的所有实例信息
         items:
           $ref: "#/definitions/MicroServiceInstance"
       schemaInfos:
         description: schema信息
         type: array
         items:
           $ref: "#/definitions/Schema"
       rules:
         type: array
         description: 该服务的所有黑白名单信息
         items:
           $ref: "#/definitions/Rule"
       providers:
         description: 所有的provider信息
         type: array
         items:
           $ref: "#/definitions/MicroService"
       consumers:
         description: 所有的consumer信息
         type: array
         items:
           $ref: "#/definitions/MicroService"
       tags:
         description: 所有的标签信息
         type: array
         items:
           $ref: "#/definitions/Tags"
       microServiceVersions:
         description: 微服务的所有版本信息，同服务名和应用名的服务的版本信息
         type: array
         items:
           type: string
  Graph:
     type: object
     properties:
       nodes:
         $ref: "#/definitions/Nodes"
  Nodes:
     type: array
     description: 图里面的节点信息
     items:
       $ref: "#/definitions/Node"
  Node:
     type: object
     properties:
       id:
         description: 微服务服务id
         type: string
       name:
         description: 微服务名字
         type: string
       appID:
         description: 应用id
         type: string
       version:
         description: 微服务版本
         type: string
       type:
         description: 类型
         type: string
       color:
         description: 颜色
         type: string
       position:
         description: 位置
         type: string
       visits:
         description: 能访问到的服务
         type: array
         items:
            type: string
  GetServicesInfoResponse:
     type: object
     properties:
       AllServicesDetail:
         description: 所有服务的相关信息
         type: array
         items:
           $ref: "#/definitions/ServiceDetail"
       statistics:
         $ref: "#/definitions/Statistics"
  Statistics:
     type: object
     description: 静态信息，包含服务个数，实例个数，有实例的服务个数，应用个数等
     properties:
       services:
         $ref: "#/definitions/StService"
       instances:
         $ref: "#/definitions/StInstance"
       apps:
         $ref: "#/definitions/StApp"
  StService:
     type: object
     properties:
       count:
         description: 微服务个数
         type: integer
       onlineCount:
         description: 有实例的微服务个数
         type: integer
  StInstance:
     type: object
     properties:
       count:
         description: 实例个数
         type: integer
  StApp:
     type: object
     properties:
       count:
         description: 应用个数
         type: integer
  GetAppsResponse:
     type: object
     properties:
       appIds:
         description: 所有应用ID
         type: array
         items:
           type: string
  getSchemaInfoResponse:
     type: object
     properties:
       schema:
         description: shema
         type: string
  MicroServiceKV:
    type: object
    properties:
      key:
        type: string
      rev:
        type: integer
      value:
        $ref: "#/definitions/MicroService"
  MicroServiceInstanceKV:
    type: object
    properties:
      key:
        type: string
      rev:
        type: integer
      value:
        $ref: "#/definitions/MicroServiceInstance"
  StringKV:
    type: object
    properties:
      key:
        type: string
      rev:
        type: integer
      value:
        type: string
  MicroserviceTagKV:
    type: object
    properties:
      key:
        type: string
      rev:
        type: integer
      value:
        $ref: '#/definitions/Properties'
  MicroserviceRuleKV:
    type: object
    properties:
      key:
        type: string
      rev:
        type: integer
      value:
        $ref: '#/definitions/Rule'
  MicroServiceDependencyRule:
    type: object
    properties:
      Dependency:
        type: array
        items:
          $ref: '#/definitions/DependencyKey'
  DependencyKV:
    type: object
    properties:
      key:
        type: string
      rev:
        type: integer
      value:
        $ref: '#/definitions/MicroServiceDependencyRule'
  Cache:
    type: object
    properties:
      services:
        type: array
        items:
          $ref: "#/definitions/MicroServiceKV"
      serviceIndexes:
        type: array
        items:
          $ref: "#/definitions/StringKV"
      serviceAliases:
        type: array
        items:
          $ref: "#/definitions/StringKV"
      serviceTags:
        type: array
        items:
          $ref: "#/definitions/MicroserviceTagKV"
      serviceRuleIndexes:
        type: array
        items:
          $ref: "#/definitions/StringKV"
      serviceRules:
        type: array
        items:
          $ref: "#/definitions/MicroserviceRuleKV"
      dependencyRules:
        type: array
        items:
          $ref: "#/definitions/DependencyKV"
      summaries:
        type: array
        items:
          $ref: "#/definitions/StringKV"
      instances:
        type: array
        items:
          $ref: "#/definitions/MicroServiceInstanceKV"
  Config:
    type: object
    properties:
      maxHeaderBytes:
        type: integer
      maxBodyBytes:
        type: integer
      readHeaderTimeout:
        type: string
      readTimeout:
        type: string
      idleTimeout:
        type: string
      writeTimeout:
        type: string
      limitTTLUnit:
        type: string
      limitConnections:
        type: integer
      limitIPLookup:
        type: string
      sslEnabled:
        type: string
      sslMinVersion:
        type: string
      sslVerifyPeer:
        type: string
      sslCiphers:
        type: string
      enablePProf:
        type: boolean
      enableCache:
        type: boolean
      selfRegister:
        type: boolean
  DumpResponse:
    type: object
    properties:
      info:
        $ref: '#/definitions/Version'
      config:
        $ref: '#/definitions/Properties'
      environments:
        $ref: '#/definitions/Properties'
      cache:
        $ref: "#/definitions/Cache"
  Clusters:
    type: object
    description: Clusters information
    additionalProperties:
      type: array
      items:
        type: string
  ClustersResponse:
    type: object
    properties:
      clusters:
        $ref: '#/definitions/Clusters'
  Error:
    type: object
    properties:
      errorCode:
        type: string
      errorMessage:
        type: string
      detail:
        type: string
