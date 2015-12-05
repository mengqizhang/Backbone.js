# Backbone.js
$(function(){
	//先建立单个模型
	var Todo = Backbone.Model.extend({
		defaults : function(){
			return {
				title:'',
				//开关的真假
				done: false
			}
		},
		// 将一个任务的完成状态置为逆状态,并保存到服务器(选中为true) 
		toggle : function(){
			this.save({done : !this.get('done')})
		}
	})
	
	var todo = new Todo;
	
	//建立一个集合
	var TodoList = Backbone.Collection.extend({
		model : Todo,
		//数据保存到本地
		localStorage: new Backbone.LocalStorage("todos-backbone"),
		//获取所有已经完成的任务数组（遍历每个model，返回包含所有通过真值检测的元素值）
		done : function(){
		 return	this.filter(function(todo){return todo.get('done')})
		},
		//获取任务列表中未完成的任务数组(返回一个删除所有values值后的 array副本)
		remaining: function() {
		  return this.without.apply(this, this.done());
		}
		/* remaining: function() {
			  return this.where({done: false});
			  return _.where(this.model,{done:false});
			},*/
	})
	var todos = new TodoList
	
	//建立单个模型所对应的视图
	var TodoView = Backbone.View.extend({
		//把template模板中获取到的html代码放到这标签中。如果不写默认为div
		tagName:'li',
		template : _.template($('#item-template').html()),
		
		events : {
			"dblclick .view" : 'edit',
			'click .destroy' : 'clear',
			'blur .edit'     : 'clos',
			'click .toggle'  : 'toggleDone'
		},
		//当初始化时，要重新渲染页面
		initialize : function(){
			//每次更新模型后重新渲染
			this.listenTo(this.model,'change',this.render);
			// remove是view中的方法，用来清除页面中的dom
			this.listenTo(this.model,'destroy',this.remove);
		},
		//当新增一个model或者修改时都会触发这个函数，进行重新渲染
		render : function(){
			// 渲染todo中的数据到 item-template 中，然后加载到页面上的li上，然后返回对自己的引用this
		 	this.$el.html(this.template(this.model.toJSON()));
			//switch值为true时,只添加class;为false时,只删除class.
			this.$el.toggleClass('done',this.model.get('done'));
			this.$el.find('input .edit').val(this.model.get('title'))
			return this;
		},
		//修改所对应的视图条目的样式
		edit : function(){
			this.$el.addClass('editing');
			this.$('.edit').focus()
		},
		//从本地存储中删除模型
		clear : function(){
			this.model.destroy()
		},
		clos  : function(){
			var value = this.$('.edit').val();
			if(!value){
				this.clear()
			}else{
				//y要把改变的数据保存到本地之后才能使模型中的数据已发生改变，所以会触发change事件,就会把修改的内容同步到界面 
				this.model.save({title:value})
				//如果没有引入本地存储就使用： this.model.set(title:value) 使数据发生改变从而再次走render
				this.$el.removeClass('editing');
			}
			
		},
		//从dom中移除这个视图
		remove : function(){
			this.$el.remove()
		},
		//点击时执行toggle（）函数，让状态值进行改变并保存到服务器中，此时会触发change事件
		toggleDone: function(){
			this.model.toggle()
		}
	})
	
	//建立整个视图
	var AppView = Backbone.View.extend({
		el : '#todoapp',
		statsTemplate : _.template($('#stats-template').html()),
		events : {
			'click #btn' : 'creatOne',
			"click #clear-completed": "clearCompleted",
      		"click #toggle-all": "toggleAllComplete"
		},
		initialize : function(){
			this.input = this.$("#new-todo");
			
			this.listenTo(todos,'add',this.addOne);
			this.listenTo(todos,'reset',this.addAll);
			this.listenTo(todos,'all',this.render);
			
			this.footer = this.$('footer');
            this.main = $('#main');
			this.allCheckbox = this.$el.find("#toggle-all")[0];
			todos.fetch();
		},
		creatOne : function(){
			var value = this.input.val();
			if(!value) {return};
			//在集合中创建一个模型的新实例，创建一个模型将立即触发集合上的add事件
			todos.create({'title':this.input.val()});//可以属性实例化一个模型
			this.input.val('')
			
		},
		addOne : function(todo){
		//此时的view对象是调用的单个模型对应的view对象，当增加一个model时就会把一个model的数据拿出来与视图结合，并且调用视图下面的render()函数重新渲染
		//并不是直接把view视图添加过去，而是把view视图下的el即li标签的内容添加过去
			var view = new TodoView({model:todo});
			this.$('#todo-list').append(view.render().el)
		},
		addAll : function(){
			todos.each(this.addOne);
		},
		render: function() {
		  var done = todos.done().length;
		  var remaining = todos.remaining().length;
		  //集合包含的模型数量
		  if (todos.length) {
			this.main.show();
			this.footer.show();
			this.footer.html(this.statsTemplate({done: done, remaining: remaining}));
		  } else {
			this.main.hide();
			this.footer.hide();
		  }
		  //若remaining为0 则说明被全选了，!0 相当于 true,若有一个没有被选中，则都为false
		  this.allCheckbox.checked = !remaining; 
		},
		//在集合上的每个执行了done方法为true的model，执行destroy方法
		clearCompleted: function() {
		  _.invoke(todos.done(), 'destroy');
		  return false;
		},
		//所有的input框的checked都随着最顶层删除所有的框的checked值保持一致，
		toggleAllComplete: function () {
		  var done = this.allCheckbox.checked;
		  todos.each(function(todo){ todo.save({done:done})})
		}
	})
	var App = new AppView;
	
})
