# 后端源代码

项目地址：[zhengqianbao-server](https://github.com/software-system-analysis-and-design/zhengqianbao-server)

代码说明：

```bash
src
│  
├─controllers
│      db_controller.go  		# 数据库核心代码
│      msg_controller.go		# 数据库管理消息模块代码
│      qFormat_controller.go	# 数据库管理问卷模块代码
│      record_controller.go		# 数据库管理用户提交的答案模块代码
│      userdb_controller.go		# 数据库管理用户信息的模块代码
├─global
│      config.go				# 全局配置文件，包含一些全局配置信息
│      
├─helper
│      response.go				# 后台返回信息的一些简单模板类
│      
├─main
│      backend.go				# 后台程序，用于生成消息通知
│      main.go					# 主程序
│      msgRounter.go			# 消息相关路由
│      qFormatRouter.go			# 问卷相关路由
│      recordRouter.go			# 用户提交答案相关路由
│      userRouter.go			# 用户信息相关路由
│      
├─models
│      answer.go				# 答案类
│      message.go				# 消息类
│      option.go				# 选项类
│      question.go				# 问题类
│      questionnaireFormat.go	# 问卷类
│      record.go				# 答案记录类
│      taskPreview.go			# 问卷简要信息类
│      user.go					# 用户类
│      
└─test
        qformat_controller_test.go	# 问卷数据库测试文件
        record_controller_test.go	# 用户提交答案数据库测试文件
        user_controller_test.go		# 用户信息数据库测试文件
```