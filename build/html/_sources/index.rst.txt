.. Lappa documentation master file, created by
   sphinx-quickstart on Wed Jul 26 15:59:06 2017.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

首页
==================

.. toctree::
   :maxdepth: 2
   :caption: Contents:

这里仅仅是Lappa开放接口的开发文档，如果需要服务端或客户端的使用方法，请查阅用户手册或帮助文档。

获取token
------------------

首先要从服务端得到secret，然后通过secret取到token，后面的每个接口都要通过header把token传到服务器才能正常调用。

地址: `http://lappa.moorol.com/api/app-token/ <http://lappa.moorol.com/api/app-token/>`_

Method: POST

Content-Type: application/x-www-form-urlencoded

传入参数说明:

=======    =======     =============
 参数名       类型            备注
=======    =======     =============
app_id     Integer      默认为0
secret     String      可到服务端获取
=======    =======     =============

若secret校验成功，会返回如下数据：


::

    {
        "token": "eyJhbGciOiJIUzI1NiIsInR5ABCDECJ9.xxxxx",
        "expires_in": 7200
    }

expires_in为token失效时间，单位为秒。当token='N/A'或expires_in=0时，意味着所用的secret已经失效，请申请新的secret.

成功获取token后，请在本地保存token，后面所有接口需要把token通过header传入，格式如下：

::

    Authorization: JWT [token]

注意JWT后面有一个半角空格

token失效后返回如下提示:

::

    {
        "detail": "Signature has expired."
    }

建议在调用接口前，先检查token是否失效，可以在本地通过expires_in来判断。请勿无必要频繁调用此接口，目前普通用户每个secret每天调用上限为100次。

上传数据
==================

新增答案和关键字
------------------

从接口新增答案

地址: `http://lappa.moorol.com/api/open/add-reply-message/ <http://lappa.moorol.com/api/open/add-reply-message/>`_

Method: POST

Content-Type: application/json

传入参数说明:

========    =======     ====================================
 参数名       类型                      备注
========    =======     ====================================
message     String      消息内容，不能为空
keywords    List        关键字集合，详细说明见下表[keywords说明]
========    =======     ====================================

keywords说明:

==============    =======     ==================================
 参数名             类型                      备注
==============    =======     ==================================
keyword           String      关键字内容，不能为空
matching_level    Integer     自定义的匹配级别，建议值：1-100
==============    =======     ==================================

示例Json:

::

    {
        "keywords": [
            {
                "keyword": "lappa",
                "matching_level": 10
            },
            {
                "keyword": "love",
                "matching_level": 5
            }
        ],
        "message": "I love lappa"
    }

返回值，1为成功，0为失败：

::

    {
        "saved": 1
    }

PS. 当HTTP状态码为201时是新增成功，200则是原来已经有了这条消息，更新成功。

新增计划发送的消息
---------------------

这些消息供服务端计划任务设置使用

地址: `http://lappa.moorol.com/api/open/add-scheduled-message/ <http://lappa.moorol.com/api/open/add-scheduled-message/>`_

Method: POST

Content-Type: application/json

传入参数说明:

================    =======     ====================================
 参数名                类型                      备注
================    =======     ====================================
message             String      消息内容，不能为空
category_number     Integer     消息类别自定义编码
================    =======     ====================================

示例Json:

::

    {
        "message": "I love lappa",
        "category_number": 10
    }

返回值，1为成功，0为失败：

::

    {
        "saved": 1
    }

PS. 当HTTP状态码为201时是新增成功，200则是原来已经有了这条消息，更新成功。

响应对话
==================

根据传入的句子，作出回复

地址: `http://lappa.moorol.com/api/save-and-reply-message/ <http://lappa.moorol.com/api/save-and-reply-message/>`_

Method: POST

Content-Type: application/x-www-form-urlencoded

传入参数说明:

================    =======     =================================================
 参数名               类型                             备注
================    =======     =================================================
msg_type            String      消息类型，暂时只能处理文本，默认为Text
message             String      传入的消息
from_username       String      来自用户名，用于回调时使用，比如微信带@的用户名，可以为空
actual_nickname     String      昵称
is_group_message    Boolean     是否是来自群聊的消息 可以为空
is_at               Boolean     是否是被@ 可以为空
group_nickname      String      群名称，可以为空
================    =======     =================================================

返回数据Json未例(普通消息)：

::

    {
        "to_username": "@@51b8d70078bded99350bf798ecc2013ae39da405521c01ef93efcd86e45f2e8a",
        "message": "虽然是出嫁了姑娘，回到老家才有归属感",
        "actual_nickname": "测试用户(ARC)"
     }

返回数据Json未例(图片消息)：

::

    {
        "to_username": "@@51b8d70078bded99350bf798ecc2013ae39da405521c01ef93efcd86e45f2e8a",
        "message": "LAPPA-IMAGE-MESSAGE http://img.027admin.com/uploads/allimg/150822/144024523I20-102501.jpg",
        "actual_nickname": "测试用户(ARC)"
    }

注：图片消息头部包含"LAPPA-IMAGE-MESSAGE"字符串，紧跟着一个空格和图片网址，请在客户端根据情况下载或直接显示图片。


定时发送
==================
.. _get_schedulers:

获取计划任务
------------------

获取计划任务列表

地址: `http://lappa.moorol.com/api/scheduler/ <http://lappa.moorol.com/api/scheduler/>`_

Method: GET

Content-Type: application/x-www-form-urlencoded

无传入参数

返回Json示例：

::

    {
        "count": 3,
        "next": null,
        "previous": null,
        "results": [
                {
                    "id": 2,
                    "trigger_time": "2017-07-24 04:00",
                    "is_every_day": false,
                    "is_every_week": true,
                    "days": "WE|TH|FR"
                },
                {
                    "id": 3,
                    "trigger_time": "2017-07-24 03:00",
                    "is_every_day": false,
                    "is_every_week": false,
                    "days": null
                },
                {
                    "id": 1,
                    "trigger_time": "2017-07-20 09:00",
                    "is_every_day": true,
                    "is_every_week": false,
                    "days": null
                }
        ],
    }

说明：

- 当is_every_week为true时，星期几days才有效
- 当is_every_day为true时，is_every_week无意义
- 当is_every_week和is_every_day都为false，表示只执行一次任务

获取计划发送的消息
------------------

获取计划任务列表

地址: `http://lappa.moorol.com/api/get-scheduled-message/ <http://lappa.moorol.com/api/get-scheduled-message/>`_

Method: POST

Content-Type: application/json

传入参数说明:

================    =======     ============================================
 参数名                类型                     备注
================    =======     ============================================
scheduler_id        Integer     计划任务ID :ref:`获取计划任务 <get_schedulers>`
================    =======     ============================================

示例Json:

::

    {
        "scheduler_id": 100
    }


返回值Json示例（普通消息）：

::

    {
        "to_username": "@bc1f5f5e5dd14eaaefd09300b304fcbb99b78b9b65b2cb8250d2dc429eadcd31",
        "message": "来啊来吧",
        "to_nickname": "王二麻子"
    }

返回值Json示例（图片消息）：

::

    {
        "to_username": "@bc1f5f5e5dd14eaaefd09300b304fcbb99b78b9b65b2cb8250d2dc429eadcd31",
        "message": "LAPPA-IMAGE-MESSAGE http://img.027admin.com/uploads/allimg/150822/144024523I20-102501.jpg",
        "to_nickname": "王二麻子"
    }

注：图片消息头部包含"LAPPA-IMAGE-MESSAGE"字符串，紧跟着一个空格和图片网址，请在客户端根据情况下载或直接显示图片。

建议或讨论
==========

QQ群：569325326