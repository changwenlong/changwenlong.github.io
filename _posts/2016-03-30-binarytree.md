---
layout: post
title:  "二叉树"
date:   2016-03-30
author:  
categories: 算法编程
excerpt: 低版本IE的bug和兼容性，点击空块级元素时
---

* content
{:toc}

## 介绍

二叉树是每个节点最多有两个子树的树结构。通常子树被称作“左子树”（left subtree）和“右子树”（right subtree）。二叉树常被用于实现二叉查找树和二叉堆。

二叉排序树（Binary Sort Tree）又称二叉查找树（Binary Search Tree），二叉查找树的左子树的所有节点都小于根结点，右子树的所有节点都大于根结点，左右子树也分别是二叉查找树，且所以节点的值不等。

二叉堆是一种特殊的堆，二叉堆是完全二元树（二叉树）或者是近似完全二元树（二叉树）。二叉堆有两种：最大堆和最小堆。最大堆：父结点的键值总是大于或等于任何一个子节点的键值；最小堆：父结点的键值总是小于或等于任何一个子节点的键值。

## 数据结构

    public class TreeNode {
        public int val;
        public TreeNode left;
        public TreeNode right;
    
        public TreeNode(int x) {
            val = x;
        }
    }

## 常见算法

二叉树常见的算法有：

1. 二叉树的前序、中序和后序遍历（递归与非递归实现）
2. 二叉树的层次遍历
3. 打印二叉树的所有路径
4. 二叉堆用来实现堆排序
5. 通过前序和中序遍历构造二叉树

### 二叉树的前序、中序和后序遍历（递归实现）

    //递归实现
    public void order(TreeNode root,List<Integer> list){
        if(root==null){
            return list;
        }
        list.add(root.val);//前序遍历
        preOrder1(root.left,list);
        //list.add(root.val);//中序遍历
        preOrder1(root.right,list);
        //list.add(root.val);//后序遍历    
    }

    //非递归实现，前序遍历
    public static ArrayList<Integer> preorder(TreeNode root) {
        ArrayList<Integer> list = new ArrayList<Integer>();
        if (root == null) {
            return list;
        }
        Stack<TreeNode> stack = new Stack<TreeNode>();
        while (!stack.isEmpty()||root!=null) {
            while (root != null) {
                list.add(root.val);
                stack.add(root);
                root = root.left;
            }
             if(!stack.isEmpty()){
                 TreeNode node = stack.pop();
                root = node.right;        
            }
        }
        return list;
    }
    
    //非递归实现，中序遍历
    public static ArrayList<Integer> inorder(TreeNode root) {
        ArrayList<Integer> list = new ArrayList<Integer>();
        if (root == null) {
            return list;
        }
        Stack<TreeNode> stack = new Stack<TreeNode>();
        while (!stack.isEmpty()||root!=null) {
            while (root != null) {
                stack.add(root);
                root = root.left;
            }
             if(!stack.isEmpty()){
                 TreeNode node = stack.pop();
                list.add(node.val);
                root = node.right;        
            }
        }
        return list;
    }

    //非递归实现，后序遍历
    public static ArrayList<Integer> postorder(TreeNode root) {
        ArrayList<Integer> list = new ArrayList<Integer>();
        if (root == null) {
            return list;
        }
        Stack<TreeNode> stack = new Stack<TreeNode>();        
        TreeNode lastVisited = null;
        while (!stack.isEmpty()||root!=null) {
            while (root != null) {
                stack.add(root);
                root = root.left;
            }
             if(!stack.isEmpty()){
                 TreeNode node = stack.peek();
                TreeNode right = node.right;
                if (right == null || right == lastVisited) {
                    list.add(stack.pop().val);
                    lastVisited = node;
                }else{
                    root = node.right;                    
                }
            }
        }
        return list;
    }

### 二叉树的层次遍历

    //借助队列实现层次遍历
    public List<Integer> levelOrder(TreeNode root) {
        List<Integer> list = new ArrayList<Integer>();
        if(root==null){
            return list;
        }
        Queue<TreeNode> q = new LinkedList<TreeNode>();
        q.add(root);
        while(!q.isEmpty()){
            int size = q.size();
            for(int i=0;i<size;i++){
                TreeNode node = q.poll();
                list.add(node.val);
                if(node.left!=null){
                    q.add(node.left);
                }
                if(node.right!=null){
                    q.add(node.right);
                }
            }
        }
        return list;
    }

### 打印二叉树的所有路径

    //resultList保存二叉树的所有路径
    private void path(TreeNode node, ArrayList<List<Integer>> resultList, ArrayList<Integer> list) {
        if (node == null) {
            return;
        }
        list.add(node.val);
        if (node.left == null && node.right == null) {        
                resultList.add(new ArrayList<Integer>(list));
        } else {
            path(node.left,  resultList, list);
            path(node.right,  resultList, list);
        }
        //什么时候remove元素需要特别注意
        list.remove(list.size() - 1);
    }




