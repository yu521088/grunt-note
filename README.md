# 前提：
1. nodejs
2. 安装grunt-cli (`npm install -g grunt-cli`)


# 项目文件：
1. package.json 用于存储npm项目的元数据， 可以在此文件中列出项目依赖的grunt和Grunt插件，放置于devDependencies配置段内。
		生成方式：
			一、 `npm init`命令创建一个基本的package.json
			二、 复制已有的package.json
			三、 用grunt-init模板创建
2. Gruntfile.js 用来配置或定义任务(task)并加载Grunt插件的。在项目根目录，和package.json在同一级目录， 并和项目源码一起加入源码管理器。

# 安装Grunt和grunt插件：
`npm install <module> --save-dev` 此命令将安装<module>， 并自动添加到package.json文件的devDependencies字段内， 例如：

	npm install grunt --save-dev
	npm install grunt-contrib-jshint --save-dev
	
# Gruntfile的构成
* “wrapper”函数
* 项目与任务配置
* 加载grunt插件和任务
* 自定义任务

实例：

	module.exports = function(grunt) {				//wrapper函数， nodejs的普通模块写法
	  // Project configuration.
	  grunt.initConfig({						//项目配置与任务
	    pkg: grunt.file.readJSON('package.json'), 
	    uglify: {							//uglify任务  (task)
	      options: {
	        banner: '/*! <%= pkg.name %> <%= grunt.template.today("yyyy-mm-dd") %> */\n'
	      },
	      build: {							//uglify的目标 (target)
	        src: 'src/<%= pkg.name %>.js',
	        dest: 'build/<%= pkg.name %>.min.js'
	      }
	    }
	  });
	
	  // 加载包含 "uglify" 任务的插件。
	  grunt.loadNpmTasks('grunt-contrib-uglify');			//加载插件
	
	  // 默认被执行的任务列表。
	  grunt.registerTask('default', ['uglify']);			//自定义任务
	
	};

# Grunt配置
Grunt的配置都是在Gruntfile的grunt.initConfig方法中指定的，主要是以任务名称命名的属性，也可以是其他任意数据。一但这些代表任意数据的属性与任务所需的属性相冲突，就将被忽略。此外，由于本身是JavaScript，你可以在这里使用任何有效的JavaScript。

## options属性(可选)
在一个任务配置中，options属性可以用来覆盖任务的默认属性值；此外，每一个目标(target)还可以拥有options属性。target级别的options将覆盖task级别的options属性。

## 文件(声明任务应该操作那些文件)
Grunt提供了强大的抽象层用于声明任务应该操作那些文件。有好几种定义src-dest(源文件-目标文件)文件映射的方式，均提供了不同程度的描述和控制操作方式。任何一种多任务（multi-task）都能理解下面的格式，所以你只需要选择满足你需求的格式就行。所有的文件格式都支持src和dest属性，简洁模式和文件数组格式还支持一些额外的属性，如： filter, nonull, dot, matchBase, expand 等
1. 简洁模式： 
	`src:['src/aa.js', 'src/bb.js'], dest: 'dest/cc.js'`
2. 文件对象模式： 这种形式支持每个目标对应多个src-dest形式的文件映射，属性名就是目标文件，源文件就是它的值(源文件列表则使用数组格式声明)。
		
	files: {
		'dest/a.js': ['src/aa.js', 'src/aaa.js'],
		'dest/a1.js': ['src/aa1.js', 'src/aaa1.js'],
	}

3. 文件数组模式： 这种形式支持每个目标对应多个src-dest文件映射，同时也允许每个映射拥有额外属性
		
	files: [
		{src: ['src/bb.js', 'src/bbb.js'], dest: 'dest/b/', nonull: true},
		{src: ['src/bb1.js', 'src/bbb1.js'], dest: 'dest/b1/', filter: 'isFile'},
	]

4. 通配符模式： 
	+ '*'匹配任意数量的字符，除了'/'
	+ '?'匹配单个字符，除了'/'
	+ '**'匹配任意数量的字符，包括'/'
	+ '{}'或表达式， 用逗号分割
	+ '!'，非表达式

5. 动态构建文件：当你希望处理大量的单个文件时，这里有一些附加的属性可以用来动态的构建一个文件列表。这些属性都可以用于简洁和文件数组模式。

expand 设置为true用于启用下面的选项：
	+ 'cwd'  源文件的相对路径
	+ 'src'  源文件相对于cwd的匹配模式
	+ 'dest' 目标文件路径的前缀
	+ 'ext'  替换目标文件的扩展名
		
实例：

	grunt.initConfig({
	  uglify: {
		static_mappings: {
		  // Because these src-dest file mappings are manually specified, every
		  // time a new file is added or removed, the Gruntfile has to be updated.
		  files: [
			{src: 'lib/a.js', dest: 'build/a.min.js'},
			{src: 'lib/b.js', dest: 'build/b.min.js'},
			{src: 'lib/subdir/c.js', dest: 'build/subdir/c.min.js'},
			{src: 'lib/subdir/d.js', dest: 'build/subdir/d.min.js'},
		  ],
		},
		dynamic_mappings: {
		  // Grunt will search for "**/*.js" under "lib/" when the "uglify" task
		  // runs and build the appropriate src-dest file mappings then, so you
		  // don't need to update the Gruntfile when files are added or removed.
		  files: [
			{
			  expand: true,     // Enable dynamic expansion. 打开动态构建
			  cwd: 'lib/',      // Src matches are relative to this path. 相对路径
			  src: ['**/*.js'], // Actual pattern(s) to match. 文件匹配模式
			  dest: 'build/',   // Destination path prefix. 目标文件前缀
			  ext: '.min.js',   // Dest filepaths will have this extension. 替换的文件扩展名
			},
		  ],
		},
	  },
	});

# 模板
使用<% %>分隔符指定的模板会在任务从它们的配置中读取相应的数据时将自动扩展扫描。
1. `<%= prop.subprop %>` 将会自动展开配置信息中的prop.subprop的值
2. `<% %>` 执行任意内联的JavaScript代码
	
# 导入外部数据
Grunt有`grunt.file.readJSON`和`grunt.file.readYAML`两个方法分别用于引入JSON和YAML数据

实例：

	grunt.initConfig({
	  pkg: grunt.file.readJSON('package.json'),		//通过grunt.file.readJSON方法引入JSON数据
	  uglify: {
	    options: {
	      banner: '/*! <%= pkg.name %> <%= grunt.template.today("yyyy-mm-dd") %> */\n'
	    },
	    dist: {
	      src: 'src/<%= pkg.name %>.js',
	      dest: 'dist/<%= pkg.name %>.min.js'
	    }
	  }
	});
