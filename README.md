#####
1.下载npm包 2.lcj i 初始化项目
###
【1】npm init 初始化脚手架文件 
【2】初始化之后，在package.json中把相应的开发依赖写入
    "dependencies": {
        "chalk": "^3.0.0",
        "child_process": "^1.0.2",
        "commander": "^4.1.1",
        "inquirer": "^7.0.4"
    },
【3】npm i 将相关的npm包下载到node_modules中
###
定义相关命令 --在bin文件夹中创建scav.js文件用于定义相关命令
【4】首先将定义脚手架的文件路径，将相关的依赖包引入，定义脚手架的版本
    #!/usr/bin/env node --harmony
    'use strict'
    // 定义脚手架的文件路径，__dirname是当前文件所在的路径
    process.env.NODE_PATH = __dirname + '/../node_modules/'

    const program = require('commander')

    // 获取package.json中的version来做为项目的版本号 
    program.version(require('../package').version)
    // 定义脚手架的用法，在program.help方法中会显示
    program.usage('<command>')
【5】给脚手架定义初始化的命令
    /*
    command为执行的命令
    description为命令的描述
    alias为简写
    action为命令相应的操作
    */
    program
        .command('init')
        .description('init a vue-based project')
        .alias('i')
        .action(()=>{
            console.log('我是初始化的方法')
        })
    
    // program.parse(arguments)会处理参数，没有被使用的选项会被存放在program.args数组中
    program.parse(process.argv)
【6】在根目录下执行下面的命令
    node bin/lcj init
【7】为了在全局使用，在package.json中添加bin相关配置
    "bin": {
        "lcj": "bin/lcj.js"
    },
【8】然后使用npm link(执行该命令)链接到全局，就可以直接在命令行使用lcj init了
【9】在最后添加命令没有被定义的操作
    if (program.args.length) {
        program.help()
    }
【10】在根目录下创建一个command文件夹用来存放命令相关操作的文件，创建一个init.js来写初始化相关操作
【11】接下来编写init.js，引入相关的依赖包，导出一个方法供lcj.js调用
    // init.js
    const inquirer = require('inquirer')
    const chalk =require('chalk')
    const {exec} = require('child_process')
    chalk.level = 3 // 设置chalk等级为3

    module.exports = ()=>{
        console.log(chalk.green('开始初始化文件'))
        console.log(chalk.gray('初始化中...'))
        console.log(chalk.green('初始化完成'))
    }
【12】在init.js文件中调用init.js文件
    program
        .command('init')
        .description('init a vue-based project')
        .alias('i')
        .action(()=>{
            require('../command/init.js')()
        })
【13】接下来使用inquirer来提问用户项目名，如果项目名为空，则给出提示并让用户重新输入
    module.exports = ()=>{
        console.log(chalk.green('开始初始化文件'))
        inquirer.prompt([{
            type:'input', // 问题类型为填空题
            message:'your  projectName:', // 问题描述
            name:'projectName', // 问题对应的属性
            validate:(val)=>{ // 对输入的值做判断
                if(val===""){
                    return chalk.red('项目名不能为空，请重新输入')
                }
                return true
            }
        }]).then(answer=>{
            console.log(chalk.gray('初始化中...'))
            console.log(chalk.green('初始化完成'))
        })
    }
【14】接下来通过child_process中的exec来执行git clone命令
    module.exports = ()=>{
        inquirer.prompt([{
            type:'input', // 问题类型为填空题
            message:'your  projectName:', // 问题描述
            name:'projectName', // 问题答案对应的属性，用户输入的内容被存储在then方法中第一个参数对应的该属性中
            validate:(val)=>{ // 对输入的值做判断
                if(val===""){
                    return chalk.red('项目名不能为空，请重新输入')
                }
                return true
            }
        }]).then(answer=>{ // 通过用户的输入进行各种操作
            console.log(chalk.green('开始初始化文件\n'))
            console.log(chalk.gray('初始化中...'))
            const gitUrl = 'https://github.com/Daisylcj/lcj-cli-vue.git'
            exec(`git clone ${gitUrl} ${answer.projectName}`,(error,stdout,stderr)=>{
                if (error) { // 当有错误时打印出错误并退出操作
                    console.log(chalk.red(error))
                    process.exit()
                }
                console.log(chalk.green('初始化完成'))
                process.exit() // 退出这次命令行操作
            })
        })
    }
【15】修改生成的文件
但是打开package.json，发现name是vue-temp，这可不符合我们的预期，所以我们要引入一个fs模块，通过fs模块来读写这个package.json，将之前输入的项目名写入，通过process.cwd()来获取当前命令行执行的路径，代码如下

    const fs = require('fs')
    module.exports = ()=>{
        inquirer.prompt([{
            type:'input', // 问题类型为填空题
            message:'your  projectName:', // 问题描述
            name:'projectName', // 问题答案对应的属性，用户输入的内容被存储在then方法中第一个参数对应的该属性中
            validate:(val)=>{ // 对输入的值做判断
                if(val===""){
                    return chalk.red('项目名不能为空，请重新输入')
                }
                return true
            }
        }]).then(answer=>{ // 通过用户的输入进行各种操作
            console.log(chalk.green('开始初始化文件\n'))
            console.log(chalk.gray('初始化中...'))
            const gitUrl = 'https://github.com/QZEming/vue-temp.git'
            exec(`git clone ${gitUrl} ${answer.projectName}`,(error,stdout,stderr)=>{ // 克隆模板并进入项目根目录
                if (error) { // 当有错误时打印出错误并退出操作
                    console.log(chalk.red('拷贝文件失败'))
                    process.exit()
                }
                fs.readFile(`${process.cwd()}/${answer.projectName}/package.json`,(err,data)=>{
                    if(error){
                        console.log(chalk.red('读取文件失败'))
                        process.exit()
                    }
                    data= JSON.parse(data.toString())
                    data.name = answer.projectName
                    fs.writeFile(`${process.cwd()}/${answer.projectName}/package.json`,JSON.stringify(data,"","\t"),err=>{
                        if(err){
                            console.log(chalk.red('写入文件失败'))
                            process.exit()
                        }
                        console.log(chalk.green('初始化完成'))
                        process.exit() // 退出这次命令行操作
                    })
                })
            })
        })
    }

