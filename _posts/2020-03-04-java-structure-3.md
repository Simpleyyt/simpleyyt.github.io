---
layout: post
categories: 数据结构
title: 面试官提到的 AVL 树，到底是个啥
tags: 
  - 炸鸡可乐
---

了解过平衡二叉树的朋友们，对它一定有印象，今天阿粉就与大家一起深入了解一下AVL树！

<!--more-->
## 一、摘要
在上篇文章，我们详细的介绍了二叉树的算法以及代码实践，我们知道不同的二叉树形态结构，对查询效率也会有很大的影响，**尤其是当树的形态结构变成一个链条结构的时候，查询最后一个元素的效率极底**，如何解决这个问题呢？

**关键在于如何最大限度的减小树的深度，从而提高查询效率**，正是基于这一点，平衡二叉查找树出现了！

平衡二叉查找树，算法由`Adel'son-Vel'skii`和 `Landis`两位大神发明，同时也俗称`AVL 树`，来自两位大神的姓名缩写，特性如下：

* **它的左子树和右子树都是平衡二叉树;**
* **且它的左子树和右子树的深度之差的绝对值（平衡因子 ） 不超过1；**

简单的说，就是为了保证平衡，当前节点的左子树、右子树的高度差不超过1！

废话也不多说了，直奔主题，算法思路如下！
## 二、算法思路
平衡二叉查找树的查找思路，与二叉树是一样，每次查询的时候对半分，只查询一部分，以达到提供效率的目的，插入、删除也一样，最大的不同点：**每次插入或者删除之后，需要计算节点高度，然后按需进行调整！**

**如何调整呢？主要方法有：左旋转、右旋转！**

下面我们分别来分析一下插入、删除的场景调整。
### 2.1、插入场景
我们来分析一下插入的场景，如下：
#### 场景一
当我们在`40`的左边或者右边插入的时候，也就是`50`的左边，只需绕`80`进行右旋转，即可达到树高度差不超过1！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-structure-3/01.jpg)

#### 场景二
当我们在`60`的左边或者右边插入的时候，也就是`50`的右边，需要进行两次旋转，先会绕`50`左旋转一次，再绕`80`右旋转一次，即可达到树高度差不超过1！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-structure-3/02.jpg)

#### 场景三
当我们在`120`的左边或者右边插入的时候，也就是`90`的右边，只需绕`80`进行左旋转，即可达到树高度差不超过1！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-structure-3/03.jpg)

#### 场景四
当我们在`85`的左边或者右边插入的时候，也就是`90`的左边，需要进行两次旋转，先会绕`90`右旋转一次，再绕`80`左旋转一次，即可达到树高度差不超过1！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-structure-3/04.jpg)

#### 总结
对于插入这种操作，总共其实只有这四种类型的插入，即：**单次左旋转、单次右旋转、左旋转-右旋转、右旋转-左旋转**，总结如下：

* **当插入节点位于需要旋转节点的左节点的左子树时，只需右旋转；**
* **当插入节点位于需要旋转节点的左节点的右子树时，需要左旋转-右旋转；**
* **当插入节点位于需要旋转节点的右节点的右子树时，只需左旋转；**
* **当插入节点位于需要旋转节点的右节点的左子树时，需要右旋转-左旋转；**

### 2.2、删除场景
接下来，我们分析一下删除场景！

其实，删除场景跟二叉树的删除思路是一样的，不同的是需要调整，删除的节点所在树，需要层层判断节点的高度差是否大于1，如果大于1，就进行左旋转或者右旋转！

#### 场景一
当删除的节点，只有左子树时，直接将左子树转移到上层即可！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-structure-3/05.jpg)

#### 场景二
当删除的节点，只有右子树时，直接将右子树转移到上层即可！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-structure-3/06.jpg)

#### 场景三
当删除的节点，有左、右子树时，因为当前节点的左子树的最末端的右子树或者当前节点的右子树的最末端的左子树，最接近当前节点，找到其中任意一个，然后将其内容替换并移除最末端节点，即可实现删除！

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-structure-3/07.jpg)

#### 总结
第三种场景稍微复杂了一些，但基本都是这么一个套路，与二叉树不同的是，删除之后需要判断树高，对超过1的进行调整，类似上面插入的左旋转、右旋转操作！

## 三、代码实践
接下来，我们从代码层面来定义一下树的实体结构，如下：
```java
public class AVLNode<E extends Comparable<E>> {

    /**节点关键字*/
    E key;

    /**当前节点树高*/
    int height;

    /**当前节点的左子节点*/
    AVLNode<E> lChild = null;

    /**当前节点的右子节点*/
    AVLNode<E> rChild = null;

    public AVLNode(E key) {
        this.key = key;
    }

    @Override
    public String toString() {
        return "AVLNode{" +
                "key=" + key +
                ", height=" + height +
                ", lChild=" + lChild +
                ", rChild=" + rChild +
                '}';
    }
}
```
我们创建一个算法类`AVLSolution`，完整实现如下：
```java
public class AVLSolution<E extends Comparable<E>> {

    /**定义根节点*/
    public AVLNode<E> root = null;

    /**
     * 插入
     * @param key
     */
    public void insert(E key){
        System.out.println("插入[" + key + "]:");
        root = insertAVL(key,root);
    }

    private AVLNode insertAVL(E key, AVLNode<E> node){
        if(node == null){
            return new AVLNode<E>(key);
        }
        //左子树搜索
        if(key.compareTo(node.key) < 0){
            //当前节点左子树不为空,继续递归向下搜索
            node.lChild = insertAVL(key,node.lChild);
            //出现不平衡，只会是左子树比右子树高，大于1的时候，就进行调整
            if(getHeight(node.lChild) - getHeight(node.rChild) == 2){
                if(key.compareTo(node.lChild.key) < 0){
                    //如果插入的节点位于当前节点的左节点的左子树，进行单次右旋转
                    node = rotateRight(node);
                }else{
                    //如果插入的节点位于当前节点的左节点的右子树，先左旋转再右旋转
                    node = rotateLeftRight(node);
                }
            }
        }else if(key.compareTo(node.key) > 0){
            //当前节点右子树不为空,继续递归向下搜索
            node.rChild = insertAVL(key,node.rChild);
            //出现不平衡，只会是右子树比左子树高，大于1的时候，就进行调整
            if(getHeight(node.rChild) - getHeight(node.lChild) == 2){
                if(key.compareTo(node.rChild.key) < 0){
                    //如果插入的节点位于当前节点的右节点的左子树，先右旋转再左旋转
                    node = rotateRightLeft(node);
                }else{
                    //如果插入的节点位于当前节点的右节点的右子树，进行单次左旋转
                    node = rotateLeft(node);
                }
            }
        } else{
            //key已经存在，直接返回
        }
        //因为节点插入，树高发生变化，更新节点高度
        node.height = updateHeight(node);
        return node;
    }

    /**
     * 删除
     * @param key
     */
    public void delete(E key){
        root = deleteAVL(key,root);
    }

    private AVLNode deleteAVL(E key, AVLNode<E> node){
        if(node == null){
            return null;
        }
        if(key.compareTo(node.key) < 0){
            //左子树查找
            node.lChild = deleteAVL(key,node.lChild);
            //可能会出现，右子树比左子树高2
            if (getHeight(node.rChild) - getHeight(node.lChild) == 2){
                node = rotateLeft(node);
            }
        } else if(key.compareTo(node.key) > 0){
            //右子树查找
            node.rChild = deleteAVL(key, node.rChild);
            //可能会出现，左子树比右子树高2
            if (getHeight(node.lChild) - getHeight(node.rChild) == 2){
                node = rotateRight(node);
            }
        }else{
            //找到目标元素，删除分三种情况
            //1.当前节点没有左子树，直接返回当前节点右子树
            //2.当前节点没有右子树，直接返回当前节点右子树
            //3.当前节点有左子树、右子树的时候，寻找当前节点的右子树的最末端的左子树，进行替换和移除
            if(node.lChild == null){
                return node.rChild;
            }
            if(node.rChild == null){
                return node.lChild;
            }
            //找到当前节点的右子树的最末端的左子树，也就是右子树最小节点
            AVLNode<E> minLChild = searchDeleteMin(node.rChild);
            //删除最小节点，如果高度变化，进行调整
            minLChild.rChild = deleteMin(node.rChild);
            minLChild.lChild = node.lChild;//将当前节点的左子树转移到最小节点上

            node = minLChild;//覆盖当前节点
            //因为是右子树发生高度变低，因此可能需要调整
            if(getHeight(node.lChild) - getHeight(node.rChild) == 2){
                node = rotateRight(node);
            }
        }
        node.height = updateHeight(node);
        return node;
    }

    /**
     * 搜索
     * @param key
     * @return
     */
    public AVLNode<E> search(E key){
        return searchAVL(key, root);
    }

    private AVLNode<E> searchAVL(E key, AVLNode<E> node){
        if(node == null){
            return null;
        }
        //左子树搜索
        if(key.compareTo(node.key) < 0){
            return searchAVL(key, node.lChild);
        }else if(key.compareTo(node.key) > 0){
            return searchAVL(key, node.rChild);
        } else{
            //key已经存在，直接返回
            return node;
        }
    } 

    /**
     * 查找需要删除的元素
     * @param node
     * @return
     */
    private AVLNode<E> searchDeleteMin(AVLNode<E> node){
        if (node == null){
            return null;
        }
        while (node.lChild != null){
            node = node.lChild;
        }
        return node;
    }

    /**
     * 删除元素
     * @param node
     * @return
     */
    private AVLNode<E> deleteMin(AVLNode<E> node){
        if(node == null){
            return null;
        }
        if (node.lChild == null){
            return node.rChild;
        }
        //移除最小节点
        node.lChild = deleteMin(node.lChild);
        //此时移除的是左节点，判断是否平衡高度被破坏
        if(getHeight(node.rChild) - getHeight(node.lChild) == 2){
            //进行调整
            node = rotateLeft(node);
        }
        return node;

    }

    /**
     * 单次左旋转
     * @param node
     * @return
     */
    private AVLNode<E> rotateLeft(AVLNode<E> node){
        System.out.println("节点：" + node.key + "，单次左旋转");
        AVLNode<E> x = node.rChild;//获取旋转节点的右节点
        node.rChild = x.lChild;//将旋转节点的右节点的左节点转移，作为旋转节点的右子树
        x.lChild = node;//将旋转节点作为旋转节点的右子树的左子树

        //更新调整节点高度(先调整旋转节点node)
        node.height = updateHeight(node);
        x.height = updateHeight(x);
        return x;
    }

    /**
     * 单次右旋转
     * @return
     */
    private AVLNode<E> rotateRight(AVLNode<E> node){
        System.out.println("节点：" + node.key + "，单次右旋转");
        AVLNode<E> x = node.lChild;//获取旋转节点的左节点
        node.lChild = x.rChild;//将旋转节点的左节点的右节点转移，作为旋转节点的左子树
        x.rChild = node;//将旋转节点作为旋转节点的左子树的右子树

        //更新调整节点高度(先调整旋转节点node)
        node.height = updateHeight(node);
        x.height = updateHeight(x);
        return x;
    }

    /**
     * 左旋转-右旋转
     * @param node
     * @return
     */
    private AVLNode<E> rotateLeftRight(AVLNode<E> node){
        System.out.println("节点：" + node.key + "，左旋转-右旋转");
        //先对当前节点的左节点进行左旋转
        node.lChild = rotateLeft(node.lChild);
        //再对当前节点进行右旋转
        return rotateRight(node);
    }

    /**
     * 右旋转-左旋转
     * @param node
     * @return
     */
    private AVLNode<E> rotateRightLeft(AVLNode<E> node){
        System.out.println("节点：" + node.key + "，右旋转-左旋转");
        //先对当前节点的右节点进行右旋转
        node.rChild = rotateRight(node.rChild);
        return rotateLeft(node);

    }

    /**
     * 获取节点高度，如果为空，等于-1
     * @param node
     * @return
     */
    private int getHeight(AVLNode<E> node){
        return node != null ? node.height: -1;
    }

    /**
     * 更新节点高度
     * @param node
     * @return
     */
    private int updateHeight(AVLNode<E> node){
        //比较当前节点左子树、右子树高度，获取节点高度
        return Math.max(getHeight(node.lChild), getHeight(node.rChild)) + 1;
    }

    /**
     * 前序遍历
     * @param node
     */
    public void frontTreeIterator(AVLNode<E> node){
        if(node != null){
            System.out.println("key:" + node.key);
            frontTreeIterator(node.lChild);//遍历当前节点左子树
            frontTreeIterator(node.rChild);//遍历当前节点右子树
        }
    }

    /**
     * 中序遍历
     * @param node
     */
    public void middleTreeIterator(AVLNode<E> node){
        if(node != null){
            middleTreeIterator(node.lChild);//遍历当前节点左子树
            System.out.println("key:" + node.key);
            middleTreeIterator(node.rChild);//遍历当前节点右子树
        }
    }

    /**
     * 后序遍历
     * @param node
     */
    public void backTreeIterator(AVLNode<E> node){
        if(node != null){
            backTreeIterator(node.lChild);//遍历当前节点左子树
            backTreeIterator(node.rChild);//遍历当前节点右子树
            System.out.println("key:" + node.key);
        }
    }

}
```
测试代码，如下：
```java
public class AVLClient {

    public static void main(String[] args) {
        //创建一个Integer型的数据结构
        AVLSolution<Integer> avlTree = new AVLSolution<Integer>();

        //插入节点
        System.out.println("========插入元素========");
        avlTree.insert(new Integer(100));
        avlTree.insert(new Integer(85));
        avlTree.insert(new Integer(120));
        avlTree.insert(new Integer(60));
        avlTree.insert(new Integer(90));
        avlTree.insert(new Integer(80));
        avlTree.insert(new Integer(130));
        System.out.println("========中序遍历元素========");

        //中序遍历
        avlTree.middleTreeIterator(avlTree.root);
        System.out.println("========查找key为100的元素========");

        //查询节点
        AVLNode<Integer> searchResult = avlTree.search(120);
        System.out.println("查找结果：" + searchResult);
        System.out.println("========删除key为90的元素========");

        //删除节点
        avlTree.delete(90);
        System.out.println("========再次中序遍历元素========");

        //中序遍历
        avlTree.middleTreeIterator(avlTree.root);
    }
}
```
输出结果如下：

![](http://www.justdojava.com/assets/images/2019/java/image_zjkl/java-structure-3/08.jpg)

## 四、总结
平衡二叉树查找树，俗称`AVL`树，**在查询的时候，操作与普通二叉查找树上的查找操作相同；插入的时候，每一次插入结点操作最多只需要单旋转或双旋转；如果是动态删除，删除之后必须检查从删除结点开始到根结点路径上的所有结点的平衡因子，也就是高度差，如果超过1就需要调整，最多可能需要`O（logN）`次旋转。**

整体上来说，平衡二叉树优于普通二叉查找树！

## 五、参考
1、[简书 - nicktming - 二叉平衡树](https://www.jianshu.com/p/22c00b3731f5)

2、[iteye - Heart.X.Raid - 平衡二叉查找树 [AVL]](https://www.iteye.com/blog/hxraid-609949)
