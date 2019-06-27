## 前端设计文档

> 主要负责人：邱奕浩 邵梓硕
>
> 审核人：邱晓裕，蒲其刚

## 一、 技术选型及理由

本项目的前端选用的技术和使用理由如下：

* **React前端开发框架** 
  * **React 框架是当前最流行也最好用的前端框架利器之一**。在github上有很高的热度（13w+ stars)。该技术有较好的成熟度，开发风险较低，在多个技术场景下都有相对成熟的解决方案，并且社区活跃，遇到难题可以及时求助得到解决。基于此，我们可以快速地进行软件的迭代开发。
  * **React速度较快，性能较优**。和其他框架相比较，React采取了一种特立独行的操作DOM的方式——不直接操作DOM，而是引入虚拟DOM的概念，其逻辑安插在 Javascript 和 真实DOM之间，能有效地提高Web性能。
  * **组件模块化开发，可维护性较高**。React是基于组件的概念编程。本质是一个单页应用，内部嵌套着多个组件。我们可以单独将某个组件模块化，编写独立的UI组件。当某个组件发生问题时，方便进行隔离和调试。
* **Material UI组件库**
  * material UI组件库是被广泛应用的一个高效美观的React组件库，其设计风格是基于`Material Design`规范，能帮助我们快速开发出美观操作简洁的web页面。
  * 官方给出material UI组件的详细教程，降低了学习成本，缩短了界面实现的时间。
* **react-router 路由框架** 
  * react-router是为React框架所定制的一套完整好用的路由解决方案，技术较为成熟，搭配React可以帮助我们快速了开发出多页面应用。
  * react-router官方教程简洁明了，方便快速上手和使用。
* **Nginx**
  * Nginx 是一个高性能且轻量级的HTTP和反向代理的服务器，因其稳定性，功能丰富，低系统资源消耗而闻名。基于此，我们使用Nginx来处理浏览器HTTP请求，返回页面文件给用户。
* **Docker**
  * docker具有持续集成，可移值，可隔离和安全等优点，方便持续部署和测试。在本项目中我们使用docker的一个容器工具docker-compose，搭配nginx快速进行前端页面的云服务器部署。



## 二、 架构设计 

项目的架构设计如下图所示，该项目使用经典的CS架构，使用Nginx进行反向代理，提供网页文件服务，并使用多个dockers运行后台程序，docker与数据库之间进行数据交互，可以很方便提高服务器的抗压能力、性能。

![](https://raw.githubusercontent.com/JoshuaQYH/blogImage/master/img/20190627142213.png)


## 三、 模块划分

本项目的前端模块划分如下：

```C++
src: 
 |--- assets       // 素材文件夹
 |--- components   // 低层组件文件夹
 |--- views        // 子界面文件夹
 |--- layouts      // 主界面文件夹
 |--- variables    // 常用的全局常量和函数
 |--- route.js     // 定义项目的基本路由
 |--- index.js     // 项目入口文件
```

这里简单介绍一下组件的划分，组件的架构大致如下图所示：

![](https://raw.githubusercontent.com/JoshuaQYH/blogImage/master/img/%E7%BB%84%E4%BB%B6%E6%9E%B6%E6%9E%84.png)

* layout：主界面。将调用下层的view组件，组成完整的界面。
* view：子界面。依赖于下层的各个子组件，构成一个个具有交互功能的子界面。
* component：子组件。接受view的调用，完成各个子组件行为的渲染。

## 四、软件设计技术

### 1. 面向对象编程

在ES6语法中，React组件开发常常使用类的形式来组织组件，以面向对象的方式来定义组件的内容和行为。

### 2. 函数式组件编程

使用函数来开发组件已经渐渐地成为了越来越多React开发者的选择。得益于 Hooks 功能，函数组件也能实现近似类组件一样的生命周期功能和React状态控制，使用Hooks已经渐渐地成为了一种编程范式。我们搭配Hooks功能，使用函数式组件开发，能够有效降低组件的状态复杂度，便于组件的解耦和组织。

在本次项目，同样也使用了函数式组件的开发理念，搭配常用的Hooks函数如useState，useEffect，帮助我们快速开发应用。

以下给出几个使用函数式组件开发的例子，加以说明。

使用 useState hook函数代替类函数中的state变量，简化了编程。

```javascript
// src/component/ShortAnswerCard/ShortAnswerCard.jsx
function ShortAnswerCard(props) {
  const { classes, content, warning, callback } = props;
  const [input, setInput] = React.useState("");

  const handleChange = event => {
    setInput(event.target.value);
    callback(event.target.value);
  };

  return (
    <Card className={classes.card}>
      <CardContent>
        <Typography className={classes.title} variant="h5" component="h2">
          {content.title}
        </Typography>
        <TextField
          id="outlined-textarea"
          label="Answer"
          placeholder="Placeholder"
          multiline
          fullWidth
          className={classes.textField}
          margin="normal"
          variant="outlined"
          onChange={handleChange}
          error={error}
          helperText={warning && error ? "回答不可为空" : null}
        />
      </CardContent>
    </Card>
  );
}
```

使用 useEffect hook 函数进行模拟类组件的生命周期函数。

```javascript
// src/view/TaskList/TaskList.jsx.... 
function TaskList(props) {

  //  模拟 componentDidMount
  React.useEffect(() => {
    const fetchData = async () => {
      const requestOptions = {
        method: "GET"
      };
      fetch(apiUrl + "/questionnaire/previews", requestOptions)
        .then(handleResponse)
        .then(response => {
          response = response.reverse();
          setTaskContent(response);
        });
    };
    fetchData(); // 请求数据
  }, []);

  return (
    <div>
      {taskContent.map(
        item =>
          item.inTrash !== 1 &&
          item.creator === localStorage.getItem("userID") && (
            <TaskContent
              taskName={item.taskName}
              taskID={item.taskID}
              taskType={item.taskType}
              money={item.money}
            />
          )
      )}
    </div>
  );
}
```

以上两种Hooks函数是在项目开发中常用的两个钩子，基于这两个接口函数，我们在大多数情况下可以编写像类组件一样的函数组件，简化组件的开发。

使用了函数式编程的组件还有以下列表，这里不做展开赘述。

```javascript
src/view/TaskList/TaskList.jsx
src/view/TaskList/Tasksquare.jsx
src/view/TaskList/TaskBoard.jsx
src/view/TaskList/TaskArray.jsx
src/view/TaskList/QuestionPage.jsx
src/view/TaskList/Questionnaire.jsx
src/view/TaskList/Notification.jsx
src/view/TaskList/CreateTask.jsx
src/view/TaskList/Commission.jsx
....
src/component/*/*jsx
```



### 3. 容器组件和表现组件分离

React 组件通常包含杂合在一起的逻辑和表现。这里采用了React一种强大而简洁的模式，成为容器组件与表现组件，按照这种模式创建的组件，可以帮助我们分离逻辑和表现这两个关注点。

我们通常在容器组件中定义获取数据逻辑并存储，然后通过props传递数据给表现组件，表现组件接受数据进行渲染呈现UI。在这个模式中，每个组件都被拆分成若干个小组件，每一个组件都有各自清晰的职责。

这里以获取问卷页面为例。

```javascript
/// src/views/QuestionPage
import SingleChoiceCard from "../../components/SingleChoiceCard/SingleChoiceCard";
import MultiChoiceCard from "../../components/MultiChoiceCard/MultiChoiceCard";
import ShortAnswerCard from "../../components/ShortAnswerCard/ShortAnswerCard";

// 容器组件 QuesitonPage
function QuestionPage(props) {
  // ...............
  const fetchQuestion = questionID => () => {
    const apiUrl = "https://littlefish33.cn:8080/questionnaire/select";
    const requestOption = {
      method: "POST",
      headers: {
        Accept: "application/json",
        "content-type": "application/x-www-form-urlencoded"
      },
      body: parseParams({ id: questionID })
    };

    fetch(apiUrl, requestOption)
      .then(handleResponse)
      .then(response => {
        setMoney(parseInt(response.money));
        setQuestionData(response.chooseData);
      });
  };

  React.useEffect(fetchQuestion(match.params.taskID), []);
    
  const createQuestionCard = (elem, index) => {
      // 渲染表现组件
      case 1:
        content = { ...content, ["title"]: elem.title };
        ret = (
          <ShortAnswerCard
            content={content}
            warning={warning && qdata[index].required}
            callback={setAns(index)}
          />
        );
        break;
	  // ......  其他表现组件
    }
    return ret;
  };
  return (
    <div>
      {qdata.map(createQuestionCard)}
    </div>
  );
}
```

容器组件和表现组件分离的设计方法，在以下文件同样应用到。

```javascript
src/views/Quesionnaire/Quesionnaire.jsx
src/views/TaskList/TaskList.jsx
src/views/Quesionnaire/Quesionnaire.jsx
src/views/TaskSquare/TaskSquare.jsx
src/views/TaskArray/TaskArray.jsx
...
```





## 五、界面UI设计



## 六、 代码编程规范

