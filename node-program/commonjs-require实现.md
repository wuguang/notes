```javascript
// let a = require('xxxx')
// 1.读取给定路径的文件里的内容，拿到字符串
//    -- 可能给了后缀，直接读
//    -- 可能没有给后缀，没有的话就按照顺序 .js .json .node优先级来加载文件
// 2.拿到绝对路径去缓存里查找一下，是否有加载过
//    -- 有就直接return
//    -- 没有创建这个模块，模块里有个this.exports对象
// 3.放如缓存


let path = require('path')
let fs = require('fs')
let vm = require('vm')
 
function Module(fullPath) {
    this.fullPath = fullPath;
    this.exports = {};
}
 
Module._extentions = ['.js', '.json', '.node'];
Module._cache = {};
 
Module._resovleFilename = function(relativePath){
    let p = path.resolve(__dirname, relativePath);
    let ext = path.extname(p);
    if (ext && Module._extentions[ext]) {
        // 写的是全路径
    } else {
        // 拼接
        for (let i = 0; i < Module._extentions.length; i++) {
            let fullPath = p + Module._extentions[i];
            try{
                fs.accessSync(fullPath);
                return fullPath
            }catch(e){
                console.log(e)
            }
        }
    }
}
Module.prototype.load = function(){
    // js加闭包，json直接当对象
    let ext = path.extname(this.fullPath)
    Module._extentions[ext](this);
}
Module.wrapper = ["(function(exports, module, require){" , "\n})"]
Module.wrap = function(script) {
    return Module.wrapper[0] + script +  Module.wrapper[1];
}
Module._extentions['.js'] = function(module){
    let codeText = fs.readFileSync(module.fullPath);
    let fnStr = Module.wrap(codeText)
    let fn = vm.runInThisContext(fnStr);
    fn.call(module.exports, module.exports, module, req)
}
Module._extentions['.json'] = function(module){
    let codeText = fs.readFileSync(module.fullPath);
    module.exports = JSON.parse(codeText)
}
function req(p) {
    // 搞出绝对路径
    let fullPath = Module._resovleFilename(p);
    // 拿到绝对路径去缓存里找
    if(Module._cache[fullPath]) {
      return Module._cache[fullPath];
    }
    // 没有缓存说明没有加载过
    let module = new Module(fullPath);
    module.load();
    Module._cache[fullPath] = module.exports;
    return module.exports
}
 
let a = req('./a')
console.log(a);
 
a.name = 'jack'
let bb = req('./a')
console.log(bb)
 
 
/* ***********引用传递，原来的那个会被改掉哦************ */
let a = require('./a')
console.log(a);//{ name: 1, age: 2 }
 
 
a.name = 'jack'
let bb = require('./a')
console.log(bb)//{ name: 'jack', age: 2 }
```
