> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.52pojie.cn](https://www.52pojie.cn/thread-1611798-1-1.html)

> [md]@[TOC](【JS 逆向系列】某方数据获取，proto 入门) 样品网址：aHR0cHM6Ly93d3cud2FuZmFuZ2RhdGEuY29tLmNuL2luZGV4Lmh0bWw = 打开网站后，......

<table cellspacing="0" cellpadding="0"><tbody><tr><td><p>@<a rel="nofollow noopener" href="【JS逆向系列】某方数据获取，proto入门">TOC</a></p><p>样品网址：aHR0cHM6Ly93d3cud2FuZmFuZ2RhdGEuY29tLmNuL2luZGV4Lmh0bWw=</p><p>打开网站后，随便搜索一个关键词，这里以【百度】为例，本次需要分析的是【SearchService.SearchService/search】这个接口</p><p><img class="" src="https://img-blog.csdnimg.cn/9ba470ac0acd451ab2113f5d6e8c74ad.png#pic_center"></p><p>这里可以看到，请求体不再是表单或者 json，而是一堆类似乱码的东西，再看看响应体</p><p><img class="" src="https://img-blog.csdnimg.cn/2da1cdb4fa9149d3a377f036b2d2d2a1.png?"><br>也是一堆乱码，有的人可能就会想，会不会是有加密呢？当然不排除这个可能。接着再看看请求头，其中有一行是【content-type: application/grpc-web+proto】，这里指明了请求体的类型是 proto。</p><p>所以这里的乱码并不是有加密，只是用 proto 这种格式序列化而已。那么接下来的流程就是编写 proto 文件，然后用 protoc 编译为对应语言可以调用的类文件。</p><p>首先下载 protoc，到 https://github.com/protocolbuffers/protobuf / 下载最新的发行版工具，我这里下载的是 win'64 版本</p><p><img class="" src="https://img-blog.csdnimg.cn/e9832322650e4b1a8c12869577c2c295.png?"></p><p>下载解压后，将里面的 bin 目录添加到环境变量，然后在 cmd 窗口输入【protoc --version】，出现版本好即为成功</p><p><img class="" src="https://img-blog.csdnimg.cn/7002b7003313401fa7a2a0cab6f03390.png#pic_center"></p><p>因为我们分析的是【SearchService.SearchService/search】这个接口，所以先下一个 XHR 断点，刷新网页，在断点处断下，返回调用堆栈的上一层</p><p><img class="" src="https://img-blog.csdnimg.cn/d1c5d6f658f74775806683270d0e1512.png?"><br><img class="" src="https://img-blog.csdnimg.cn/6bad3c30750a4b3bb614b8636cd00050.png?" style="width: auto;"></p><p>看到一个类似组包的函数，那么在前面加一个断点，再次刷新</p><p><img class="" src="https://img-blog.csdnimg.cn/526cb98139e0489f9a7c7ccc0d986ba0.png?"><br>接着进入【r.a】</p><p><img class="" src="https://img-blog.csdnimg.cn/60c03cf2d3ec41c1bcad8413ef170d59.png?" style="width: auto;"><br>发现这是一个 webpack 打包的 js，并且所有的信息序列化与反序列化的操作都在这个 js 里面，一般情况下，都是用的标准库的工具，所以首先直接搜索【.deserializeBinaryFromReader = 】，为什么搜索这个呢？这就好比 json 的数据会搜索【JSON.】是一样的。</p><p><img class="" src="https://img-blog.csdnimg.cn/cbaf25b589b34614b7dce385c5bc631e.png?" style="width: auto;"></p><p>这里就获取到每个信息是如何解析的，也就是可以获取信息的结构</p><p>如果一个一个来写的话，那就有点麻烦了，而且还怕会出错，那么为了保证准确性，所以这次使用 ast 来生成 proto 文件，首先吧这个【app.1d44779a.js】下载到本地，并且执行下面代码</p><pre>const parser = require("@babel/parser");
// 为parser提供模板引擎
const template = require("@babel/template").default;
// 遍历AST
const traverse = require("@babel/traverse").default;
// 操作节点，比如判断节点类型，生成新的节点等
const t = require("@babel/types");
// 将语法树转换为源代码
const generator = require("@babel/generator");
// 操作文件
const fs = require("fs");

//定义公共函数
function wtofile(path, flags, code) {
&nbsp; &nbsp; var fd = fs.openSync(path,flags);
&nbsp; &nbsp; fs.writeSync(fd, code);
&nbsp; &nbsp; fs.closeSync(fd);
}

function dtofile(path) {
&nbsp; &nbsp; fs.unlinkSync(path);
}

var file_path = 'app.1d44779a.js';
var jscode = fs.readFileSync(file_path, {
&nbsp; &nbsp; encoding: "utf-8"
});

// 转换为AST语法树
let ast = parser.parse(jscode);
let proto_text = `syntax = "proto2";\n\n// protoc --python_out=. app_proto2.proto\n\n`;

traverse(ast, {
&nbsp; &nbsp; MemberExpression(path){
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;if(path.node.property.type === 'Identifier' &amp;&amp; path.node.property.name === 'deserializeBinaryFromReader' &amp;&amp; path.parentPath.type === 'AssignmentExpression'){
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;let id_name = path.toString().split('.').slice(1, -1).join('_');
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;path.parentPath.traverse({
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; VariableDeclaration(path_2){
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;if(path_2.node.declarations.length === 1){
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;path_2.replaceWith(t.expressionStatement(
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; t.assignmentExpression(
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;"=",
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;path_2.node.declarations[0].id,
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;path_2.node.declarations[0].init
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; )
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;))
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;}
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; },
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; SwitchStatement(path_2){
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;for (let i = 0; i &lt; path_2.node.cases.length - 1; i++) {
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;let item = path_2.node.cases[i];
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;let item2 = path_2.node.cases[i + 1];
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;if(item.consequent.length === 0 &amp;&amp; item2.consequent[1].expression.type === 'SequenceExpression'){
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; item.consequent = [
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;item2.consequent[0],
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;t.expressionStatement(
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;item2.consequent[1].expression.expressions[0]
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;),
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;item2.consequent[2]
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; ];
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; item2.consequent[1] = t.expressionStatement(
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;item2.consequent[1].expression.expressions[1]
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; )
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;}else if(item.consequent.length === 0){
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; item.consequent = item2.consequent
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;}else if(item.consequent[1].expression.type === 'SequenceExpression'){
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; item.consequent[1] = t.expressionStatement(
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;item.consequent[1].expression.expressions[1]
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; )
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;}
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;}
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; }
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;});
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;let id_text = 'message ' + id_name + ' {\n';
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;let let_id_list = [];
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;for (let i = 0; i &lt; path.parentPath.node.right.body.body[0].body.body[2].cases.length; i++) {
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; let item = path.parentPath.node.right.body.body[0].body.body[2].cases[i];
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; if(item.test){
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;let id_number = item.test.value;
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;let key = item.consequent[1].expression.callee.property.name;
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;let id_st, id_type;
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;if(key.startsWith("set")){
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;id_st = "optional";
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;}else if(key.startsWith("add")){
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;id_st = "repeated";
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;}else{
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;// map类型，因为案例中用不到，所以这里省略
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;continue
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;}
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;key = key.substring(3, key.length);
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;id_type = item.consequent[0];
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;if(id_type.expression.right.type === 'NewExpression'){
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;id_type = generator.default(id_type.expression.right.callee).code.split('.').slice(1).join('_');
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;}else{
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;switch (id_type.expression.right.callee.property.name) {
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; case "readString":
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;id_type = "string";
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;break;
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; case "readDouble":
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;id_type = "double";
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;break;
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; case "readInt32":
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;id_type = "int32";
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;break;
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; case "readInt64":
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;id_type = "int64";
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;break;
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; case "readFloat":
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;id_type = "float";
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;break;
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; case "readBool":
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;id_type = "bool";
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;break;
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; case "readPackedInt32":
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;id_st = "repeated";
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;id_type = "int32";
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;break;
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; case "readBytes":
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;id_type = "bytes";
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;break;
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; case "readEnum":
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;id_type = "readEnum";
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;break;
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; case "readPackedEnum":
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;id_st = "repeated";
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;id_type = "readEnum";
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;break;
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;}
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;}
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;if(id_type === 'readEnum'){
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;id_type = id_name + '_' + key + 'Enum';
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;if(let_id_list.indexOf(id_number) === -1){
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; id_text += '\tenum ' + id_type + ' {\n';
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; for (let j = 0; j &lt; 3; j++) {
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;id_text += '\t\t' + id_type + 'TYPE_' + j + ' = ' + j + ';\n';
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; }
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; id_text += '\t}\n\n';
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; id_text += '\t' + id_st + ' ' + id_type + ' ' + key + ' = ' + id_number + ';\n';
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; let_id_list.push(id_number)
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;}
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;}else{
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;if(let_id_list.indexOf(id_number) === -1){
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; id_text += '\t' + id_st + ' ' + id_type + ' ' + key + ' = ' + id_number + ';\n';
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; let_id_list.push(id_number)
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;}
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;}
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp; }
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;}
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;id_text += '}\n\n';
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;&nbsp; &nbsp;proto_text += id_text
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;}
&nbsp; &nbsp; }
});

wtofile('app_proto2.proto', 'w', proto_text);
</pre><p>运行后可以得到一个【app_proto2.proto】的文件，开后发现有少量报错</p><p><img class="" src="https://img-blog.csdnimg.cn/bca0fa7f873a415f99bb2fa2e09adfdc.png?"><br>在网页中搜索这个信息结构</p><p><img class="" src="https://img-blog.csdnimg.cn/0d41007daa054ba4aaf501c8963043df.png?"></p><p>这里的 o 省略了路径名，所以无法获取到完整路径就报错了，手动补充一下即可，往上查找 o 的来源</p><p><img class="" src="https://img-blog.csdnimg.cn/444ab27602fc4acc923545a8657c1456.png?"></p><p>o 是来自于【e348】，那么搜索这个</p><p><img class="" src="https://img-blog.csdnimg.cn/d4e76de50fc04808bd89951a9f36ca94.png?"><br>一直拉到最下面看看导出的名称是什么</p><p><img class="" src="https://img-blog.csdnimg.cn/9c4327b245f24e3bb17774c1f63edbd1.png?"></p><p>接着补全一下路径</p><p><img class="" src="https://img-blog.csdnimg.cn/ecf7e6da75164fe1ac667e3288dab11c.png?"></p><p>其他的报错都可以如此类推解决，然后在当前目录打开 cmd，输入指令编译出 python 可调用的类</p><pre>protoc --python_out=. app_proto2.proto
</pre><p>此时就可以在当前目录的一个【app_proto2_pb2.py】文件</p><p>尝试使用这个生成的了进行数据序列化，使用 proto 文件前，需要先安装依赖库</p><pre>pip install protobuf
</pre><p><img class="" src="https://img-blog.csdnimg.cn/512dee8bfaa743e698bcd7b3560fdce2.png?"></p><p>但是并不是直接序列化后就可以请求，这里可以看到对请求体还有一层包装，序列化的内容被设置到偏移 5 的位置，而偏移 1 的位置设置了【l】参数，这里的【l】参数就是后面数据的长度</p><p>那么尝试按照这个格式去生成一个请求体去试试能不能获取数据</p><pre>import app_proto2_pb2
import requests_html
import struct

def main():
&nbsp; &nbsp; requests = requests_html.HTMLSession()
&nbsp; &nbsp; search_request = app_proto2_pb2.SearchService_SearchRequest()
&nbsp; &nbsp; search_request.InterfaceType = app_proto2_pb2.SearchService_SearchRequest.SearchService_SearchRequest_InterfaceTypeEnum.Value('SearchService_SearchRequest_InterfaceTypeEnumTYPE_0')
&nbsp; &nbsp; search_request.Commonrequest.SearchType = 'paper'
&nbsp; &nbsp; search_request.Commonrequest.SearchWord = '百度'
&nbsp; &nbsp; search_request.Commonrequest.CurrentPage = 1
&nbsp; &nbsp; search_request.Commonrequest.PageSize = 20
&nbsp; &nbsp; search_request.Commonrequest.SearchFilterList.append(app_proto2_pb2.SearchService_CommonRequest.SearchService_CommonRequest_SearchFilterListEnum.Value('SearchService_CommonRequest_SearchFilterListEnumTYPE_0'))
&nbsp; &nbsp; data = search_request.SerializeToString()
&nbsp; &nbsp; data = bytes([0]) + struct.pack("&gt;i", len(data)) + data
&nbsp; &nbsp; print(data)

&nbsp; &nbsp; url = 'https://s.wanfangdata.com.cn/SearchService.SearchService/search'
&nbsp; &nbsp; headers = {
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4901.0 Safari/537.36',
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;'Content-Type': 'application/grpc-web+proto',
&nbsp; &nbsp; }

&nbsp; &nbsp; response = requests.post(url, headers=headers, data=data)
&nbsp; &nbsp; print(response.content)
&nbsp; &nbsp; print(len(response.content))
</pre><p><img class="" src="https://img-blog.csdnimg.cn/60edb1873c0f44d295d1c45480063e24.png?"></p><p>看起来返回的数据是正确了，那么接着尝试去反序列化数据</p><p><img class="" src="https://img-blog.csdnimg.cn/2f083585d2134425af79fcc8a11b6da5.png?" style="width: auto;"><br>没有报错，非常好，说明编写的 proto 文件没有问题，本文结束，下面是完整代码</p><pre>import app_proto2_pb2
import requests_html
import struct

def main():
&nbsp; &nbsp; requests = requests_html.HTMLSession()
&nbsp; &nbsp; search_request = app_proto2_pb2.SearchService_SearchRequest()
&nbsp; &nbsp; search_request.InterfaceType = app_proto2_pb2.SearchService_SearchRequest.SearchService_SearchRequest_InterfaceTypeEnum.Value('SearchService_SearchRequest_InterfaceTypeEnumTYPE_0')
&nbsp; &nbsp; search_request.Commonrequest.SearchType = 'paper'
&nbsp; &nbsp; search_request.Commonrequest.SearchWord = '百度'
&nbsp; &nbsp; search_request.Commonrequest.CurrentPage = 1
&nbsp; &nbsp; search_request.Commonrequest.PageSize = 20
&nbsp; &nbsp; search_request.Commonrequest.SearchFilterList.append(app_proto2_pb2.SearchService_CommonRequest.SearchService_CommonRequest_SearchFilterListEnum.Value('SearchService_CommonRequest_SearchFilterListEnumTYPE_0'))
&nbsp; &nbsp; data = search_request.SerializeToString()
&nbsp; &nbsp; data = bytes([0]) + struct.pack("&gt;i", len(data)) + data
&nbsp; &nbsp; print(data)

&nbsp; &nbsp; url = 'https://s.wanfangdata.com.cn/SearchService.SearchService/search'
&nbsp; &nbsp; headers = {
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/101.0.4901.0 Safari/537.36',
&nbsp; &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;'Content-Type': 'application/grpc-web+proto',
&nbsp; &nbsp; }

&nbsp; &nbsp; response = requests.post(url, headers=headers, data=data)
&nbsp; &nbsp; data_len = struct.unpack("&gt;i", response.content[1:5])[0]

&nbsp; &nbsp; search_response = app_proto2_pb2.SearchService_SearchResponse()
&nbsp; &nbsp; search_response.ParseFromString(response.content[5: 5 + data_len])
&nbsp; &nbsp; print(search_response)

if __name__ == '__main__':
&nbsp; &nbsp; main()
</pre><p>附加内容：对于较少的信息结构是，直接手动写也很快。但是多的时候，手动写重复的工作多，还很容易出错，ast 的作用就体现出来了。对于 web 端可以 proto 文件自动还原可以使用 ast，而在 app 的话，那该如何解决呢？可以参考下面文章使用 frida 解决</p><p><a rel="nofollow noopener" href="https://github.com/SeeFlowerX/frida-protobuf">https://github.com/SeeFlowerX/frida-protobuf</a></p></td></tr></tbody></table>