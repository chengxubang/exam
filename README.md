springmvc实战在线考试系统
=

程序有问题联系[程序帮](http://suo.nz/530ijn)：QQ1022287044

项目介绍
----
> springmvc实战在线考试系统，学生自主注册账号，选择自己所在班级，进行在线模拟考试。 项目主要分为用户管理，资源管理，考试管理，试卷管理，作业管理，成绩管理等几个大的模块，针对每个模块划分管理员、教室、学生三种角色，给予每个不同角色相应的页面，操作逻辑以及权限。

项目适用人群
----
正在做毕设的学生，或者需要项目实战练习的Java学习者

技术选型：
-----
* 前端
  * Html/Css/JavaScript
  * Bootstrap
  * jQuery
  * UploadFive
* 后端
  * Spring/SpringMVC/Hibernate
  * Spring Security
  * slf4j/log4j
  * Gson
  * POI
  * Druid
* 数据库
  * MySQL

项目访问地址
---
```
http://localhost:8090
管理员帐号admin， 密码admin
```

项目结构
-----
![项目结构](/src/image/项目结构.png)


项目截图
----

- 注册

![注册](/src/image/注册.png)

- 管理员-公告管理

![管理员-公告管理](/src/image/管理员-公告管理.png)

- 管理员-教师管理

![管理员-教师管理](/src/image/管理员-教师管理.png)

- 教师-试卷管理

![教师-试卷管理](/src/image/教师-试卷管理.png)

- 教师-题库管理

![教师-题库管理](/src/image/教师-题库管理.png)

- 教师-作业管理

![教师-作业管理](/src/image/教师-作业管理.png)

- 学生-考试

![学生-考试](/src/image/学生-考试.png)

- 学生-考试结果

![学生-考试结果](/src/image/学生-考试结果.png)

- 学生-试题讨论

![学生-试题讨论](/src/image/学生-试题讨论.png)

- 学生-作业下载

![学生-作业下载](/src/image/学生-作业下载.png)






数据库配置：
----
```
db.driver=com.mysql.jdbc.Driver
db.url=jdbc:mysql://localhost:3306/exam?useSSL=false&characterEncoding=UTF-8
db.username=root
db.password=root123
```


具体实现:
-----
1.老师角色添加试卷
```diff
//前端试卷创建
<div class="panel-body">
  <form:form action="${ctx}/exampaper/save" method="post" cssClass="form-horizontal"
    enctype="multipart/form-data" modelAttribute="entity">
    <form:hidden path="id" />
    <div class="form-group">
      <label for="name" class="col-sm-2 control-label"> 试卷名 </label>
      <div class="col-sm-4">
        <form:input cssClass="form-control" path="name"  autocomplete="off"/>
      </div>
    </div>
    <div class="form-group">
      <label for="description" class="col-sm-2 control-label">描述</label>
      <div class="col-sm-4">
        <form:textarea cssClass="form-control" path="description" />
      </div>
    </div>
    <div class="form-group">
      <label for="content" class="col-md-2 control-label">题目文件</label>
      <div class="col-md-5">
        <input name="file" type="file" accept="application/vnd.ms-excel,
          application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"/>
      </div>
    </div>
    <div class="col-md-8 text-center">
      <button class="btn btn-primary" type="submit">提交</button>
      &emsp;&emsp;&emsp;&emsp;
      <a class="btn btn-warning" href="${ctx}/exampaper/list">返回</a>
    </div>
  </form:form>
</div>

//后端处理
public String save( @ModelAttribute("entity") ExamPaper examPaper, @RequestParam("file") MultipartFile file) { CallBackMessage msg;
    try {
        InputStream inputStream = file.getInputStream();
        List<Question> questions = ExcelToQuestionUtils.readQuestions(inputStream);
        inputStream.close();
        examPaper.setQuestions(questions);
        examPaper.setClassId(Integer.valueOf(CurrentUtils.getCurrentUser().getClassId()));
        return baseSave(examPaper);
    } catch (IOException e) {
        e.printStackTrace();
        msg = CallBackMessage.createDangerMsg("服务器异常，请重试！");
    } catch (EncryptedDocumentException e) {
        e.printStackTrace();
        msg = CallBackMessage.createDangerMsg("题目文档已被加密，无法识别！");
    } catch (InvalidFormatException e) {
        e.printStackTrace();
        msg = CallBackMessage.createDangerMsg("题目文档格式有误！");
    } catch (IllegalArgumentException e) {
        e.printStackTrace();
        msg = CallBackMessage.createDangerMsg(e.getMessage());
    }
    msg.addToCurrentSession();
    return redirect(LIST_PATH);
}
```

2. 创建考试，与试卷绑定
```diff
//前端JSP代码
<div class="col-xs-10">
  <div class="panel panel-info">
    <div class="panel-heading">
      <h3 class="panel-title">
        <span class="glyphicon glyphicon-align-justify"></span> &nbsp;考试管理
      </h3>
    </div>
    <div class="panel-body">
      <form:form action="${ctx}/exam/save" method="post"
        cssClass="form-horizontal" modelAttribute="entity">
        <form:hidden path="id" />
        <div class="form-group">
          <label for="name" class="col-sm-2 control-label">考试名称</label>
          <div class="col-sm-4">
            <form:input cssClass="form-control" path="name" autocomplete="off" />
          </div>
        </div>
        <div class="form-group">
          <label for="description" class="col-sm-2 control-label">描述</label>
          <div class="col-sm-4">
            <form:textarea cssClass="form-control" path="description" />
          </div>
        </div>
          <div class="form-group">
              <label for="description" class="col-sm-2 control-label">截止提交日期</label>
              <div class="col-sm-4">
                  <div class='input-group date' id='datetimepicker2'>
                      <form:input cssClass="form-control" path="endTime" autocomplete="off"/>
                      <span class="input-group-addon">  <span class="glyphicon glyphicon-calendar"></span> </span>
                  </div>
              </div>
          </div>
        <div class="form-group">
          <label for="description" class="col-sm-2 control-label">考试时间（分钟）</label>
          <div class="col-sm-4">
            <form:input cssClass="form-control" path="time" type="number" min="0" autocomplete="off"  />
          </div>
        </div>
        <div class="form-group">
          <label for="description" class="col-sm-2 control-label">考试试卷</label>
          <div class="col-sm-4">
            <form:select path="exampaperId" cssClass="form-control"
              items="${exampapers}" itemLabel="name" itemValue="id" />
          </div>
        </div>
        <div class="col-md-8 text-center">
          <button class="btn btn-primary" type="submit">提交</button>
          &emsp;&emsp;&emsp;&emsp;
          <a class="btn btn-warning" href="${ctx}/exam/list">返回</a>
        </div>
      </form:form>
    </div>
  </div>
</div>
//后端入库处理
public String save( @RequestParam("exampaperId") Long exampaperId,@ModelAttribute(ENTITY_ATTRIBUTE_NAME) Exam entity) {
    ExamPaper examPaper = exampaperService.findByID(exampaperId);
    examPaper.setClassId(Integer.valueOf(CurrentUtils.getCurrentUser().getClassId()));
    entity.setExampaper(examPaper);
    entity.setClassId(Integer.valueOf(CurrentUtils.getCurrentUser().getClassId()));
    return baseSave(entity);
}
```
 
3. 试卷题目
```
<div class="panel-body">
    <form:form action="${ctx}/question/save" method="post"
               cssClass="form-horizontal" modelAttribute="entity">
        <form:hidden path="id"/>
        <div class="form-group">
            <label for="type" class="col-md-2 control-label">题目类型</label>
            <div class="col-md-5">
                <form:select cssClass="form-control" path="type" disabled="true">
                    <form:option value="判断"/>
                    <form:option value="单选"/>
                    <form:option value="多选"/>
                </form:select>
            </div>
        </div>
        <div class="form-group">
            <label for="content" class="col-md-2 control-label">题干</label>
            <div class="col-md-5">
                <form:textarea cssClass="form-control" path="content" disabled="true"/>
            </div>
        </div>
        <c:forEach items="${entity.choices}" var="choice" varStatus="st">
            <div class="form-group _oldChoice">
                <input type="hidden" name="choices[${st.index}].id" value="${choice.id}"/>
                <label class="col-md-2 control-label">选项</label>
                <div class="col-md-5">
      <textarea name="choices[${st.index}].content" class="form-control" readonly><c:out value="${choice.content}" />
      </textarea>
                </div>
                <div class="col-md-2">
                    <label class="control-label">是否为正确选项</label>
                    <input type="checkbox" name="choices[${st.index}].answer"
                        ${choice.answer == true ? 'checked="checked"' : ''} disabled="true"/>
                </div>
            </div>
        </c:forEach>
        <div id="newChoiceDiv"></div>
        <div id="choiceAnswerDiv"></div>
        <div class="form-group">
            <label for="content" class="col-md-2 control-label">所属试卷</label>
            <div class="col-md-5">
                <form:checkboxes path="exampaperIds" delimiter="<br/>"
                                 items="${exampapers}" itemLabel="name" itemValue="id" disabled="true"/>
            </div>
        </div>
        <div class="col-md-9 text-center"><hr/>
            <a class="btn btn-warning" href="${ctx}/question/list">返回</a>
        </div>
    </form:form>
</div>
```

4. 学生考试作答
```diff
//前端jsp代码
<div class="container">
<div class="row">
    <!-- 题目导航 -->
    <div class="col-md-3" id="question-nav">
        <div class="panel panel-info">
            <div class="panel-heading">
                <h3 class="panel-title">
                    <span class="glyphicon glyphicon-th-list"></span>&nbsp;题目导航
                </h3>
            </div>
            <div class="panel-body row">
                <c:forEach items="${entity.exampaper.questions}" var="question" varStatus="st">
                    <div class="col-md-3 text-center" style="margin-bottom: 10px;">
                        <a class="btn btn-default btn-xs" href="#question-${question.id}">${st.count}</a>
                    </div>
                </c:forEach>
                <div class="row">
                    <div class="col-md-12 text-center">
                        <h4>剩余时间</h4>
                        <h4 id="left_time">${entity.time}分钟0秒</h4>
                    </div>
                </div>
            </div>
        </div>
    </div>
    <!-- 题目 -->
    <div class="col-md-9">
        <div class="panel panel-info">
            <div class="panel-heading">
                <h3 class="panel-title">
                    <span class="glyphicon glyphicon-th-list"></span>&nbsp;${entity.name}
                </h3>
            </div>
            <!-- 答题div -->
            <div class="panel-body" style="font-size:18px;">
                <form action="${ctx}/exam/subExam" method="post">
                    <input type="hidden" name="examId" value="${entity.id}"/>
                    <c:forEach items="${entity.exampaper.questions}" var="question" varStatus="st">
                        <c:if test="${question.id!=null}">
                            <c:if test="${st.index==0}">
                                <c:if test="${not empty list}">
                                    <div class="panel panel-primary">
                                        <div class="panel-heading">
                                            <h3 class="panel-title">简答题试卷</h3>
                                        </div>
                                        <div class="panel-body" style="font-size: 16px;">
                                            <p class="well">请自行下载简答试卷,答卷完成请及时上传</p>
                                            <p>
                                                <a class="btn btn-success pull-right _download" onclick="_download(${list[0].id});" >点击下载</a>
                                            </p>
                                        </div>
                                    </div>
                                </c:if>
                            </c:if>
                        </c:if>
                        <input type="hidden" name="questionIds[${st.index}]" value="${question.id}"/>
                        <div class="panel panel-default" id="question-${question.id}">
                            <div class="panel-heading">
                                <h3 class="panel-title">${question.type}题</h3>
                            </div>
                            <div class="panel-body _chooseDiv"
                                 data-question-type="${question.type == '多选' ? 'checkbox' : 'radio'}">
                                <p>${st.count}.&nbsp;${question.content}</p>
                                <c:forEach items="${question.choices}" var="choice" varStatus="innerSt">
                                    <div class="checkbox">
                                        <label>
                                            <input type="checkbox" value="${choice.id}"
                                                   name="chooses[${st.index}][${innerSt.index}]"/>
                                                ${int2char[innerSt.index]}.&nbsp;${choice.content}
                                        </label>
                                    </div>
                                </c:forEach>
                            </div>
                        </div>
                    </c:forEach>
                    <div class="col-md-9 text-center">
                        <hr/>
                        <button class="btn btn-primary" type="submit" id="submitButton">提交</button>
                        &emsp;&emsp;&emsp;&emsp;
                        <a class="btn btn-warning" href="${ctx}/exam/show">返回</a>
                    </div>
                </form>
            </div>
        </div>
    </div>
</div>

``` 

5. 学生成绩查询
```
<div class="container">
<div class="row">
  <div class="col-md-offset-1 col-md-10">
    <div class="panel panel-info">
      <div class="panel-heading">
        <h3 class="panel-title">
          <span class="glyphicon glyphicon-th-list"></span>&nbsp;我的成绩
        </h3>
      </div>
      <div class="panel-body">
        <%@include file="/common/show-message.jsp"%>
        <table class="table table-bordered">
          <tr>
            <th>考试名称</th>
            <th>考试试卷</th>
            <th>考试日期</th>
            <th>成绩</th>
          </tr>
          <c:forEach items="${entities}" var="result">
            <tr>
              <td><c:out value="${result.exam.name}" /></td>
              <td><c:out value="${result.exam.exampaper.name}" /></td>
                <td><fmt:formatDate type="date"  value="${result.sysModifyLog.createDate}" /></td>
              <td><c:out value="${result.grade}" /></td>
            </tr>
          </c:forEach>
        </table>
      </div>
    </div>
  </div>
</div>
</div>
```

操作流程
----
1. admin用户创建老师角色并绑定所属班级
2. 老师角色登录管理后台，上传题库并创建试卷
3. 将创建的试卷进行考试绑定，明确告知学生考试是具体那一试卷
4. 学生自行注册并选中所属班级，注册登录后能看到自己所属班级是有考试
5. 进行考试作答并提交，等待批改
6. 查看错题及成绩
7. 额外还有课件下载及作业和公告查看功能

项目后续
----
其他ssh，ssm，springboot版本后续迭代更新，请持续关注



