---
title: javascript执行上下文和执行栈
categories: js
# tags: [javascript]
---

执行上下文是当前 JavaScript 代码被解析和执行时所在环境的抽象概念。

#### 执行上下文的类型

执行上下文总共有三种类型

- **全局执行上下文**：只有一个，浏览器中的全局对象就是 window 对象，`this` 指向这个全局对象。
- **函数执行上下文**：存在无数个，只有在函数被调用的时候才会被创建，每次调用函数都会创建一个新的执行上下文。
- **Eval 函数执行上下文**： 指的是运行在 `eval` 函数中的代码，不用很少用而且不建议使用。

#### 执行栈

执行栈，也叫调用栈，具有 LIFO（后进先出）结构，用于存储在代码执行期间创建的所有执行上下文。

首次运行JS代码时，会创建一个**全局**执行上下文并Push到当前的执行栈中。每当发生函数调用，引擎都会为该函数创建一个**新的函数**执行上下文并Push到当前执行栈的栈顶。

根据执行栈LIFO规则，当栈顶函数运行完成后，其对应的**函数**执行上下文将会从执行栈中Pop出，上下文控制权将移到当前执行栈的**下一个**执行上下文。

#### 执行上下文的创建

执行上下文分两个阶段创建：**1）创建阶段；** **2）执行阶段**

#### 创建阶段

- 1、确定 **this** 的值，也被称为 **This Binding**。
- 2、**LexicalEnvironment（词法环境）** 组件被创建。
- 3、**VariableEnvironment（变量环境）** 组件被创建。

直接看伪代码可能更加直观

```javascript
ExecutionContext = {  
  ThisBinding = <this value>,     // 确定this 
  LexicalEnvironment = { ... },   // 词法环境
  VariableEnvironment = { ... },  // 变量环境
}
```

##### This Binding

- **全局**执行上下文中，`this` 的值指向全局对象，在浏览器中`this` 的值指向 `window`对象，而在`nodejs`中指向这个文件的`module`对象。
- **函数**执行上下文中，`this` 的值取决于函数的调用方式。具体有：默认绑定、隐式绑定、显式绑定（硬绑定）、`new`绑定、箭头函数，具体内容会在【this全面解析】部分详解。

##### 词法环境（Lexical Environment）

词法环境有两个**组成部分**

- 1、**环境记录**：存储变量和函数声明的实际位置
- 2、**对外部环境的引用**：可以访问其外部词法环境

词法环境有两种**类型**

- 1、**全局环境**：是一个没有外部环境的词法环境，其外部环境引用为**null**。拥有一个全局对象（window 对象）及其关联的方法和属性（例如数组方法）以及任何用户自定义的全局变量，`this` 的值指向这个全局对象。
- 2、**函数环境**：用户在函数中定义的变量被存储在**环境记录**中，包含了`arguments` 对象。对外部环境的引用可以是全局环境，也可以是包含内部函数的外部函数环境。

直接看伪代码可能更加直观

```javascript
GlobalExectionContext = {  // 全局执行上下文
  LexicalEnvironment: {          // 词法环境
    EnvironmentRecord: {           // 环境记录
      Type: "Object",                 // 全局环境
      // 标识符绑定在这里 
      outer: <null>                 // 对外部环境的引用
  }  
}

FunctionExectionContext = { // 函数执行上下文
  LexicalEnvironment: {        // 词法环境
    EnvironmentRecord: {          // 环境记录
      Type: "Declarative",         // 函数环境
      // 标识符绑定在这里               // 对外部环境的引用
      outer: <Global or outer function environment reference>  
  }  
}
```

##### 变量环境

变量环境也是一个词法环境，因此它具有上面定义的词法环境的所有属性。

在 ES6 中，**词法** 环境和 **变量** 环境的区别在于前者用于存储**函数声明和变量（ let 和 const ）**绑定，而后者仅用于存储**变量（ var ）**绑定。

使用例子进行介绍

```javascript
let a = 20;  
const b = 30;  
var c;

function multiply(e, f) {  
 var g = 20;  
 return e * f * g;  
}

c = multiply(20, 30);
```

执行上下文如下所示

```javascript
GlobalExectionContext = {

  ThisBinding: <Global Object>,

  LexicalEnvironment: {  
    EnvironmentRecord: {  
      Type: "Object",  
      // 标识符绑定在这里  
      a: < uninitialized >,  
      b: < uninitialized >,  
      multiply: < func >  
    }  
    outer: <null>  
  },

  VariableEnvironment: {  
    EnvironmentRecord: {  
      Type: "Object",  
      // 标识符绑定在这里  
      c: undefined,  
    }  
    outer: <null>  
  }  
}

FunctionExectionContext = {  

  ThisBinding: <Global Object>,

  LexicalEnvironment: {  
    EnvironmentRecord: {  
      Type: "Declarative",  
      // 标识符绑定在这里  
      Arguments: {0: 20, 1: 30, length: 2},  
    },  
    outer: <GlobalLexicalEnvironment>  
  },

  VariableEnvironment: {  
    EnvironmentRecord: {  
      Type: "Declarative",  
      // 标识符绑定在这里  
      g: undefined  
    },  
    outer: <GlobalLexicalEnvironment>  
  }  
}
```

**变量提升**的原因：在创建阶段，函数声明存储在环境中，而变量会被设置为`undefined`（在 `var` 的情况下）或保持未初始化（在 `let` 和 `const`的情况下）。所以这就是为什么可以在声明之前访问 `var` 定义的变量（尽管是 `undefined` ），但如果在声明之前访问 `let` 和 `const` 定义的变量就会提示引用错误的原因。这就是所谓的变量提升。

#### 执行阶段

此阶段，完成对所有变量的分配，最后执行代码。

如果 Javascript 引擎在源代码中声明的实际位置找不到 `let` 变量的值，那么将为其分配 `undefined` 值。