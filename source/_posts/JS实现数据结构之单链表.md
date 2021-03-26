---
title: JS实现数据结构之单链表
date: 2019-07-18 10:44:14
tags: 
	- 数据结构
	- 技能知识图谱
	- 单链表
---

```
/**
*	@decoration 新Node节点
*/
function Node(element){
	this.element = element;
	this.next = null;
}
```
<!-- more -->

```
/**
*	@decoration 单链表数据结构
*/
function NodeList(){
	this.head = new Node('head');
}


/**
* 	@decoration 末尾追加节点
* 	@param element 新节点data
*/
NodeList.prototype.appendNode = function(element){
	var currNode = this.head;
	while(currNode.next){
		currNode = currNode.next;
	}
	currNode.next = new Node(element);
	return this;
}

/**
*	@decoration 查找某个节点的上一个节点
*	@param element 要查找的节点
*/
NodeList.prototype.findPrevNode = function(element){
	var currNode = this.head;
	while(currNode.next && currNode.next.element !== element){
		currNode = currNode.next;
	}
	return currNode;
}

/**
*	@decoration 在指定节点之后插入某个节点
*   @param index 要插入的节点坐标
*	@param element 要插入的节点
*/	
NodeList.prototype.InsertNode = function(index,element){
	var 
		findCurrElement = this.findPrevNode(index),
		newEle = new Node(element);
	newEle.next = findCurrElement.next;
	findCurrElement.next = newEle;
	return this;
}

/**
*	@decoration 删除指定节点
*	@param element 要删除的节点
*/
NodeList.prototype.removeNode = function(element){
	var currNode = this.findPrevNode(element);
	if(currNode.next) currNode.next = currNode.next.next;
	return this;
}

/**
*	@decoration 展示当前所有的节点
*/
NodeList.prototype.displayNodes = function(){
	var currNode = this.head;
	while(currNode.next){
		currNode = currNode.next;
		console.log(currNode);
	}
}

/**
*	@Decoraton 单链表测试
*/
window.onload = function(){
	NodeListInstance = new NodeList();
	//插入元素
	NodeListInstance.appendNode('1');
	NodeListInstance.appendNode('2');
	NodeListInstance.appendNode('3');
	//在指定节点插入元素
	NodeListInstance.InsertNode('2','4');
	//显示所有的元素
	NodeListInstance.displayNodes();
	console.log(NodeListInstance);
}
```
