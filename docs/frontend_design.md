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

在layout，view目录下都有组件以继承React.Component的类表示：

``` c++
view:
 |--- UserProfile		//个人信息界面
 |--- LoginPage			//登录界面
 |--- RegisterPage		//注册界面
 |--- DashBoard			//DashBoard界面

layout:
 |--- Admin				//主界面跳转
```

类组件主要重写render函数，返回需要在前端渲染出的html内容，根据实际要求其他组件生命周期函数。

在类组件中`state`被识别为类的私有变量，在react组件的渲染，更新和扩展方面带来了很大的便利，以**LoginPage**为例：

##### 初始化

`state`在构造函数中初始化，我们将它视为这个页面的逻辑中所需要的一些数据或者状态量即可：

``` jsx
constructor(props) {
    super(props);

    this.state = {
      userphone: "",		//数据
      password: "",			//数据
      submitted: false,		//状态量
      responseMsg: " ",		//数据
      user_error: false,	//状态量
      password_error: false	//状态量
    };
    
    //...
}
```

##### 实时渲染

类组件将`state`中的数据放进渲染到前端的html中去，可以实时显示数据在前端。

首先在render中拿出`state`中的数据，格式为`{数据1名，数据2名，...} = this.state`:

``` jsx
render() {
    const {
      userphone,
      password,
      submitted,
      responseMsg,
      user_error,
      password_error
    } = this.state;	//可以拿到state中的所有数据和状态量
    return (
    	//...
    );
}
```

其次在组件中利用变量名即可：

``` jsx
<TextField
    ...
    value={userphone}				//将state中userphone显示在输入区域
    ...
    />
```

也可以进行逻辑判断：

``` jsx
{submitted && !userphone && (		//使用状态量进行逻辑判断
    <div className="help-block" style={{ color: "red" }}>
        手机号不可为空
    </div>
)}
```

##### 修改

类组件对变量的修改使用`this.setState()`，参数必须是和`state`相同的目录结构，比如提交表单后修改状态量：

``` jsx
handleSubmit(e) {
    e.preventDefault();
    this.setState({ submitted: true });			//修改state中的submitted为true
    //...
}
```

##### 可扩展性

由于数据变量使用的便利和类的特性，类组件拥有良好的可扩展性，比如可以在类中加入自定义方法，方便对组件逻辑进行模块化管理：

``` jsx
//文件位置：src/views/LoginPage/LoginPage.jsx
class LoginPage extends React.Component{
    
    handleChange(e) {}
    handleSubmit(e) {}
    login(userphone, password) {}
}
```

或者将数据进行结构化存入`state`中：

``` jsx
//文件位置：src/views/UserProfile/UserProfile.jsx
this.state = {
    ...
    user: {
        name: null,
        phone: null,
        password: null,
        ...
    },
    ...
};
```

因此对组件进行渲染和功能上的扩展都比较方便。

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

UI原型设计工具：[xiaopiu](https://www.xiaopiu.com/)

UI设计展示链接：[界面UI设计](https://software-system-analysis-and-design.github.io/Dashboard/docs/index.html)

## 六、 代码编程规范

### JS规范

- 使用[ES 2016+ 规范](http://kangax.github.io/compat-table/es2016plus/)

- 在ESLint的代码风格检查的基础上增加额外要求：

  - 命名同时还需要关注语义，如：
    - 变量名应当使用名词camel命名
    - boolean类型的应当使用is、has等起头,表示其类型
    - 函数名应当用动宾短语

  - 永远不省略分号
  - 组件文件和组件使用相同的名字，组件名必须避免使用Vue保留标签名（包括HTML标签和Vue内部标签）

### React/JSX 规范

#### 基本

- 原则上每个文件只写一个组件, 多个[无状态组件](https://facebook.github.io/react/docs/reusable-components.html#stateless-functions)可以放在单个文件中. eslint: [`react/no-multi-comp`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/no-multi-comp.md#ignorestateless).

- 推荐使用 JSX 语法编写 React 组件, 而不是 `React.createElement`

#### 命名

**扩展名**: React 组件使用 `.jsx` 扩展名

- **文件名**: 文件名使用帕斯卡命名. 如, `ReservationCard.jsx`
- **引用命名**: React组件名使用帕斯卡命名, 实例使用骆驼式命名. eslint: [`react/jsx-pascal-case`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-pascal-case.md)

- **组件命名**: 组件名与当前文件名一致. 如 `ReservationCard.jsx` 应该包含名为 `ReservationCard`的组件. 如果整个目录是一个组件, 使用 `index.js`作为入口文件, 然后直接使用 `index.js` 或者目录名作为组件的名称
- **属性命名**: 避免使用 DOM 相关的属性来用命名自定义属性

#### 代码缩进

- 遵循 JSX 语法缩进/格式. eslint: [`react/jsx-closing-bracket-location`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-closing-bracket-location.md)

#### 括号

- 将多行 JSX 标签写在 `()`里. eslint: [`react/jsx-wrap-multilines`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-wrap-multilines.md)

#### 标签

- 对于没有子元素的标签, 总是自关闭标签. eslint: [`react/self-closing-comp`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/self-closing-comp.md)
- 如果组件有多行的属性, 关闭标签时新建一行. eslint: [`react/jsx-closing-bracket-location`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-closing-bracket-location.md)

#### 函数/方法

- 当在 `render()` 里使用事件处理方法时, 提前在构造函数里把 `this` 绑定上去. eslint: [`react/jsx-no-bind`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-no-bind.md)
- 在React组件中, 不要给所谓的私有函数添加 `_` 前缀, 本质上它并不是私有的
- 在 `render` 方法中总是确保 `return` 返回值. eslint: [`react/require-render-return`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/require-render-return.md)

#### 生命周期顺序

- `static` 方法（可选）

- `constructor` 构造函数

- `getChildContext` 获取子元素内容

- `componentWillMount` 组件渲染前

- `componentDidMount` 组件渲染后

- `componentWillReceiveProps` 组件将接受新的数据

- `shouldComponentUpdate` 判断组件是否需要重新渲染

- `componentWillUpdate` 上面的方法返回 `true`时, 组件将重新渲染

- `componentDidUpdate` 组件渲染结束

- `componentWillUnmount` 组件将从DOM中清除, 做一些清理任务

- 事件绑定 如 `onClickSubmit()` 或 `onChangeDescription()`

- *render 里的 getter 方法* 如 `getSelectReason()` 或 `getFooterContent()`

- *可选的 render 方法* 如 `renderNavigation()` 或 `renderProfilePicture()`

- `render` render() 方法
