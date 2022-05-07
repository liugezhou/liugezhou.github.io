---
layout: post
title: mongoose官方文档总结
date: 2021-03-12
updated: 2022-05-04
description: mongoose官方文档的简单学习
toc: true
category: 
  服务端
---

<font color=blue>更新说明：对文章目录排版做了调整。</font>
<font color=blue> 更新时间：2022-05-04</font>

## 一、mongoose

> -   安装：npm install  mongoose

```javascript
// 1，引入mongoose
const mongoose = require('mongoose')
// 2. 连接本地数据库
let db = mongoose.connect('mongodb://localhost/test')

const db = mongoose.connection;
db.on('error', console.error.bind(console, 'connection error:'));
db.once('open', function() {
  // we're connected!
});
```

> mongoose里，一切始于Schema：

```javascript
let tomSchema = mongoose.Schema({
	name:String
})

//接着，把这个Schema编译成一个Model
let Tom = mongoose.model('Tom',tomSchema)

// 给这个model加一个code方法
Tom.methods.code = function(){
	let nickname = this.name ? "The programmar name is :" + this.name:'I don't have name'
  console.log(nickname)
}

//model是我们构造document的class，每个document都是一个Tom对象
let Tomliu = new Tom({name:'liugezhou'})
Tomliu.code() //The programmar name is :liugezhou

// save
Tomliu.save(function(err,item){
	if (err) return console.error(err);
   item.speak();
})

// 获取所有的Tom
Tom.find(function(err,tomlius){
	if(err) return console.error(err);
  console.log(tomlius)
})

//获取特定数据
Tom.find({name:/^liugezhou/},callback)
```

## 二、Schema-模式

> -  每个Schema都会映射到MongoDB 的collection，并定义这个collection里的文档构成
> 
语法：
> -  const shcema = mongoose.Schema({})
> 
允许使用的Schematypes有：
> -  String
> -  Boolean
> -  Date
> -  Number
> -  Array
> -  Buffer
> -  Mixed
> - ObjectId
> 
除了映射collection外，还可以定义
> -  document的instance methods
> -  model的static Model methods
> -  复合索引
> - 文档的生命周期钩子，也成为中间件

**model**
> 我们要把一个Schema转化为一个model，要使用 
> -   let model = mongoose.model(modelName,schema)  函数

**collection和document**
> -   collection相当于关系型数据库中的表
> -   document相当于一条数据，在这里有特别需要注意的一点是：
> -   collection不要求文档有相同的结构，在一个collection文档中不必具有相同的fileds，对于单个field在一个collection中的不同文档中可以是不同的数据类型

**实例方法methods**
> -   documents是model的实例,document有自带的实例方法，当然也可以自定义我们自己的方法。

```javascript
const animalSchema = mongoose.Schema({type:String,name:String})

animalSchema.methods.findSameType = function (cb){
  return this.model('Animal').find(type:this.type,cb)
}

const Animal = mongoose.model('Animal',animalSchema)
const dog = new Animal({type:'dog'})

dog.findSameType(function(err,dogs){
	console.log(dogs)
})
```
**静态方法**
> -   静态方法与实例方法的区别是，实例方法是在每个model的实例中可以访问，而静态方法是每个model直接访问

```javascript
animalSchema.statics.findByName = function(name,cb) {
	return this.find({name:new RegExp(name,'i')},cb)
}
const Animal = mongoose.model('Animal',animalSchema)
Animal.findByName('fido',function(err,animal){
	console.log(animals)
})
```
**查询助手**
> -   查询助手作用于query实例，方便定义自己的查询扩展

```javascript
  animalSchema.query.byName = function(name) {
    return this.find({ name: new RegExp(name, 'i') });
  };

  var Animal = mongoose.model('Animal', animalSchema);
  Animal.find().byName('fido').exec(function(err, animals) {
    console.log(animals);
  });
```
**索引**
> -   Mongodb支持secondary indexes,在mongoose中，我们在Schema中定义索引，索引字段级别和shcema级别

```javascript
  var animalSchema = new Schema({
    name: String,
    type: String,
    tags: { type: [String], index: true } // field level
  });

  animalSchema.index({ name: 1, type: -1 }); // schema level
```

**虚拟值 Virtual**
> - [ ]  Virtual是document的属性，但是不会保存到MongoDB，getter可以用于格式化和组合字段数据，setter可以很方便的分解一个值到多个字段。

```javascript
  // define a schema
  var personSchema = new Schema({
    name: {
      first: String,
      last: String
    }
  });

  // compile our model
  var Person = mongoose.model('Person', personSchema);

  // create a document
  var axl = new Person({
    name: { first: 'Axl', last: 'Rose' }
  });
```
> -   如果你要log出全名，可以这么做：

```
console.log(axl.name.first + ' ' + axl.name.last); // Axl Rose
```
> -   但是每次都这么拼接实在太麻烦了， 推荐你使用[virtual property getter](http://www.mongoosejs.net/docs/api.html#virtualtype_VirtualType-get)， 这个方法允许你定义一个 fullName 属性，但不必保存到数据库。

```javascript
personSchema.virtual('fullName').get(function () {
  return this.name.first + ' ' + this.name.last;
});
```
> -   现在, mongoose 可以调用 getter 函数访问 fullName 属性：

```
console.log(axl.fullName); // Axl Rose
```
> -   如果对 document 使用 toJSON() 或 toObject()，默认不包括虚拟值， 你需要额外向 [toObject()](http://www.mongoosejs.net/docs/api.html#document_Document-toObject) 或者 toJSON() 传入参数** { virtuals: true }**。
> 
你也可以设定虚拟值的 setter ，下例中，当你赋值到虚拟值时，它可以自动拆分到其他属性：

```javascript
personSchema.virtual('fullName').
  get(function() { return this.name.first + ' ' + this.name.last; }).
  set(function(v) {
    this.name.first = v.substr(0, v.indexOf(' '));
    this.name.last = v.substr(v.indexOf(' ') + 1);
  });

axl.fullName = 'William Rose'; // Now `axl.name.first` is "William"
```
> -   再次强调，虚拟值不能用于查询和字段选择，因为虚拟值不储存于 MongoDB。

**选项**
> -   Schema有很多可配置选项，你可以在构造时传入或者直接set，选项较多，暂不学习整理。

```javascript
new Schema({..}, options);

// or

var schema = new Schema({..});
schema.set(option, value);
```
## 三、SchemaTypes-模式类型

以下是mongoose的所有合法SchemaTypes：
> - String
> - Boolean
> - Number
> - Array
> - Buffer
> - Date
> - Schema.Types.ObjectId
> - Schema.Types.Mixed
> - Schema.Types.Decimal128

**SchemeType选项**
> -   你可以直接声明schema type为某一种type，或者赋值一个含有type属性的对象

```javascript
var schema1 = new Schema({
  test: String // `test` is a path of type String
});

var schema2 = new Schema({
  test: { type: String } // `test` is a path of type string
});
```

> -   除了type属性，还可以对这个字段路径指定其它属性，比如在保存之前全部转换为小写

```javascript
var shema2 = new Schema({
	test:{
  	type:String,
    lowercase:true
  }
})
```
**全部可用**
> - required:布尔值或者函数 如果值为真，为此属性添加require验证器
> - default: 任何值或函数 设置此路径默认值，如果是函数m，函数返回值为默认值
> - select: 布尔值 指定query的默认projections
> - validate: 函数校验
> - get:函数，使用Object.defineProperty()定义自定义getter
> - set：同上
> - alias：别名

**索引相关**
> 可以使用 schema type定义索引相关
> - index：布尔值    是否对这个属性创建索引
> - unique:布尔值    是否对这个属性创建唯一索引
> - sparse:布尔值    是否对这个属性创建稀疏索引

## 四、Connections-连接

> -    可以使用 mongoose.connect()连接MongoDB，默认端口27017    

**操作缓存**
> 就是说不必等待上面的connect连接成功后，就可以使用创建的 Mongoose models
> 禁用缓存，要修改 bufferCommands配置，mongoose.set('bufferCommands',fasle)

**选项**
> connect 方法也接受 options 参数，这些参数会传入底层 MongoDB 驱动。

**回调**
> connect()函数接受回调函数，或返回一个Promise

**keepAlive**
> 对于长期运行的后台应用，启用毫秒级 keepAlive 是一个精明的操作。不这么做你可能会经常 收到看似毫无原因的 "connection closed" 错误。
> mongoose.connect(uri,{keepAlive:120})

## 五、models-模型

> -   [Models](http://www.mongoosejs.net/docs/api.html#model-js) 是从 `Schema` 编译来的构造函数。 它们的实例就代表着可以从数据库保存和读取的 [documents](http://www.mongoosejs.net/docs/documents.html)。
> -   从数据库创建和读取 document 的所有操作都是通过 model 进行的。

```javascript
var schema = new mongoose.Schema({ name: 'string', size: 'string' });
var Tank = mongoose.model('Tank', schema);
```
> 上面的参数 Tank是跟model对应的集合(collection)对应的单数形式。
> Mongoose会自动找到名称是model的名字的复数形式。
> 比如上例，Tank这个model对应数据库中tanks这个collection
> .model()这个函数是对 schema做了拷贝
> 确保在调用.model()之前把所有需要的东西都加进shema里。

**构造documents**
> -   documents是model的实例，创建谈并保存到数据库非常简单：

```javascript
const Tank = mongoose.model('Tank',TankSchema)
const small = new Tank({ size:' small'})
small.save(function(err){
	if (err) return hanldeError(err) 
})

// or
Tank.create({size:'small'},function(err,small){
	if (err) return handleError(err)
})
```
**查询**
> -   查询文档可以用model的find、findbyId,findOne，和where这些静态方法。

**删除**
> -   model的remove方法可以删除所有匹配查询条件(condition)的文档

```javascript
Tank.remove({size:small},function(err){
	if(err) return handler(err)
})
```
**更新**
> -  `model` 的 `update` 方法可以修改数据库中的文档，不过不会把文档返回给应用层。
> -   如果想更新单独一条文档并且返回给应用层，可以使用 [findOneAndUpdate](http://www.mongoosejs.net/docs/api.html#model_Model.findOneAndUpdate) 方法。

## 六、文档-Documents

> -   Mongoose document代表着MongoDB文档的一对一映射。每个document都是他的Model的实例。

**更新**
使用findById:
```javascript
Tank.findById(id,function(err,tank){
	if (err) return handlerError(err)
  
  tank.size = 'large';
  //tank.set({size:'large'})
  tank.save(function(err,updateTank){
  	if (err) return handlerError(err)
    res.send(updateTank)
  })
})
```

-   若仅仅需要更新数据,而不需要获取数据再去更新：
> Tank.update({_id:id},{$set:{size:'large'}},callback)

-   更新后我们还需要返回这个文档：findByIdAndUpdate
```javascript
Tank.findByIdAndUpdate(id,{$set:{size:'large'}},{new:true},function(err,tank){
  if (err) return handlerError(err)
  res.send(tank)
})
```
## 七、子文档-SubDocuments

> 子文档是指嵌套在另一个文档中的文档。
> 在Mongoose中，意味着你可以在里嵌套另一个schema。
> Mongoose子文档有两种不同的概念：子文档数组和单个嵌套子文档

```javascript
const chidlSchema = new Schema({name:String})

const parentSchema =  new Schema({
	children:[childSchema],
  child:childSchema
})

```
> 子文档与文档的区别是 子文档不能单独保存，他们会在他们的顶级文档保存时保存。

```javascript
const Parent = mongoose.model('Parent',parentSchema)
const parent = new Parent({children:[{name:'liu'},{name:'ge'},{name:'zhou;}]})
parent.children[0].name = 'liu'       
parent.save(callback)         
```

## 八、Queries 查询

> -   Model的多个静态辅助方法都可以查询文档
> -   Query实例有一个.then()函数，用法类似Promise

我们看一下demo，查询persons表中name中属性last为Ghost值的文档，只查询 name和occupation两个字段
```javascript
const Person = mpngoose.model('Pseron',PersonSchema);

Person.findOne({name.last:'Ghost'},'name occupation',function(err,person){
	if(err) return handleError(err)
  console.log('%s %s is a %s',person.name.fisrt,person.name.last,person.occupation)
})
```
> 查询结果的格式取决于做什么操作：
> - findOne()是单个文档
> - find() 是文档列表
> - count() 是文档数量
> - update() 是更新的文档数量

## 九 中间件--Middleware

> 1. 中间件(pre 和 post 钩子)是在异步函数执行时函数传入的控制函数。    
> 1. Middleware is specified on the shema.    
> 1. Mongoose4.x有四种中间件：**doucument**中间件、**model**中间件、**aggregate**中间件、**quer**y中间件。
> 1. document 中间件支持以下document操作：    
>    1. **init**
>    1. **validate**
>    1. **save**
>    1. **remove**
> 5. query 中间件支持以下 Model 和 Query 操作
>    1. count
>    1. find
>    1. findOne
>    1. findOneAndUpdate
>    1. findOneAndRemove
>    1. updade
> 6. aggregate 中间件作用于MyModel.aggregate()，他会在你对 aggregate 对象调用 exec()时执行
>    1. aggregate
> 7. Model中间件支持以下操作：
>    1. insertMany
> 8. 所有中间件支持 pre 和 post 钩子。
> 8. Query 没有 remove()钩子，只有 docuemnt 有，如果设定了remove钩子，他将会在你调用 myDoc.remove()触发，而不是 myModel.remove(),另外，create()函数会触发 save()钩子。

**pre**
> pre钩子分为『串行』和『并行』两种

- **串行：**
> 串行中间件一个接一个的执行。具体来说，上一个中间件调用 next 的时候下一个执行

```javascript
const schema = new Schema(..);
schema.pre('save',function(next){
	// to stuff
  next()
})
```
> 在 mongoose5.x 中，除了手动调用 next 函数，还可以返回一个 Promise，甚至是 async/await。

```javascript
schema.pre('save',function(){
	return doStuff().
					then(()=> doMoreStuff())
})

// or

shcema.pre('save',async function(){
	await doStuff();
  await doMoreStuff();
})
```

- **并行**
> 并行中间件提供细粒度流控制。

```javascript
const schema = new Schema(..)

shcema.pre('save',true,function(next,done){
	next()
  setTimeout(done,100)
})
```
> 在这个例子中，save 方法将在所有中间件都调用了 done 方法的时候才会执行。
> 使用场景:
> - 复杂的数据校验
> - 删除依赖文档(删除用户后删除他的所有文档)
> - asynchronous defaults
> - asynchronous tasks that a certain action triggers


**Post**
> Post中间件在方法执行之后调用，这个时候每个 pre 中间件都已完成

```javascript
schema.post('init',function(doc){
	 console.log('%s has been initialized from the db', doc._id);
})

schema.post('validate',function(doc){
	 console.log('%s has been validated (but not saved yet)', doc._id);
})

schema.post('save',function(doc){
   console.log('%s has been saved', doc._id);
})

schema.post('remove',function(doc){
	 console.log('%s has been removed', doc._id);
})
```

- **异步 Post 钩子**
> 如果你给 post 钩子的回调函数传入两个参数，mongoose 会认为第二个参数是 next()函数，可以通过 next 触发下一个中间件

```javascript
schema.post('save',function(doc,next){
	setTimeout(function(){
  	console.log('pot1')
    next()
  },100)
})

schema.post('save', function(doc, next) {
  console.log('post2');
  next();
});
```

- **Save/Validate钩子**
> save()函数触发 validate()钩子，mongoose validate()钩子其实就是 pre('save')钩子，这意味着所有pre('validate')和 post('validate')钩子都会在 pre('save')之前调用。

- **findAndUpdate() 和 Query 中间件使用注意**
> pre 和 post save()钩子都不执行于 update()、 findOneAndUpdate()等情况 
> mongoose4.x为这些函数制定了新钩子

```javascript
schema.pre('find',function(){
	conosle.log(this instanceof mongoose.query) //true
  this.start = Date.now()
})

schema.post('find',function(result){
	conosle.log(this instanceof mongoose.query) //true
  // prints returned documents
  console.log('find() returned ' + JSON.stringify(result));
  // prints number of milliseconds the query took
  console.log('find() took ' + (Date.now() - this.start) + ' millis');
})
```
**错误处理中间件**
> `next()` 执行错误时，中间件执行立即停止。但是我们有特殊的 post 中间件技巧处理这个问题 —— 错误处理中渐渐，它可以在出错后执行你指定的代码。
> 错误处理中间件比普通中间件多一个 `error` 参数，并且 `err` 作为第一个参数传入。 而后错误处理中间件可以让你自由地做错误的后续处理

```javascript
const schema = new Schema({
	name:{
  	type:String,
    unique:true
  }
})

schema.post('save',function(err,doc,next){
  if (error.name === 'MongoError' && error.code === 11000) {
    next(new Error('There was a duplicate key error'));
  } else {
    next(error);
  }
})

Person.create([{name:'liu'},{name:'Gezhou'}]);
```
## 十、填充--Populate

**demo**
MongoDb 在 3.2之后，也有像 sql 中的 join 聚合操作，那就死$lookup,而 mongoose 拥有更强大的 populate，可以让你在别的 collection 中引用 document。
Populate 可以自动替换 document 中的指定字段，替换内容从其他 collection 获取，我们填充(populate)单个或者多个 document、单个或者多个对象，甚至是 query 返回的一切对象：
```javascript
const mongoose = require('mongoose')
const Schema = mongoose.Schema;

const personSchema = Schema({
	_id:Schema.types.ObjectId,
  name:String,
  age:Number,
  stories:[{type:Schema.types.ObjectId,ref:'Story'}]
})

const storySchema = Schema({
	author:{type:Schema.types.ObjectId,ref:'Person'},
  title:String,
  fans:[{type:Schema.types.ObjectId,ref:'Person'}]
})

const Story = mongose.model('Story',storySchema)
const Person = mongose.model('Person',personSchema)
```
> 我们创建了两个model，Person model中的 stories 字段为 ObjectID 数组，ref 选项告诉mongoose 在填充的时候使用哪个 model，上面的例子就是指 Story 的 model。所有储存在此的_id 都必须是 Story model 中的 document 的 _id

**保存 refs**
保存 refs 与保存普通属性一样，把_id的值赋给他就好了
```javascript
const author = new Person({
	_id:new mongoose.Types.objectId(),
  name:'liugezhou',
  age:18
})

author.save(function(err){
	if (err) return handleError(err);
  const story1 = new Story({
  	title:'my book',
    author:author._id
  })
  story1.save(function(err){
  	if (err) return handleError(err);
  })
})
```
**Population**
```javascript
Story.
	findOne({title:'my book'}).
	populate('author').
	exec(function(err,story){
		if (err) return handleError(err);
  	console.log('The author is %s', story.author.name);
	})

```
**设置被填充字段**
mongoose4.0之后，你可以手动填写一个字段
```javascript
Story.findOne({title:'my book'},function(err,story){
	if (err) return handleError(err);
  
  story.author = author
  console.log(story.author.name);
})
```
## 十一、鉴别器--Discriminator

Discriminator是一种 schema 继承机制。它允许你在相同的底层MongoDb collection上使用部分重叠的 schema 建立多个 model。

