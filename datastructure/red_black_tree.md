#红黑树
##红黑树定义与特性
1. 每个节点要么是红色，要么是黑色
2. 根节点为黑色
3. 红色节点的两个子节点都是黑色的，黑色节点则无要求
4. 每个叶子节点(并非元素节点，而是NULL节点)都是黑色的
5. 每个节点，从该节点到所有后代的简单路径都包含相同的黑色节点

##红黑树相关引理证明
定义：*bh(black height)*为黑色节点的高度，即从某个节点x(不包括自身)到一个叶子节点(NULL)的高度，由红黑树定义可知从该节点到任一一个叶子节点都包含相同的黑色节点，一般说bh黑高为根节点的黑高。

1. 任一节点x为根的子树至少包含(2^bh(x))-1个内部节点，其中bh(x)为以x为根的树的黑色节点高度。
可以使用归纳法证明：
假设x的高度为bh=0，则x为树的叶子节点(NULL)，则x的内部节点为(2^bh(x))-1 = (2^0)-1=0，结论成立;
对于任一高度的内部节点x，其高度为bh(x) 当x.color=red时成立,或bh(x)-1，当x.color=balck时成立，对于两个子节点(NULL也为子节点之一)每个子节点至少有(2^(bh(x)-1))-1，因此以x为跟的子树至少包含的内部节点数分别为：
		节点数n>=左节点数+右节点数+根节点=((2^(bh(x)-1))-1)+((2^(bh(x)-1))-1)+1=(2^bh(x))-1
得证。

2. 一颗有n个内部节点的红黑树的高度h至多为2lg(n+1)，即 h<=2lg(n+1)
设树的高度为h，与前面定义的黑高bh不一样，根据特性3得知从根节点(不包括根)到任一叶子节点中，至少有一半以上的节点为黑色节点，即bh(root)>=h(root)/2；于是有
		n>=2^(bh/2)-1>=2^(h/2)-1，化简得: h<=2lg(n+1)

3. 从某个节点x都其叶子节点的全部简单路径中，最长的路径至多为最短的路径的两倍，红黑树是近似平衡的二叉树。
假设x最长的路径为(a1,a2.....an)
由红黑树性质(红色节点的两个子节点必须是黑色)，得知这条路径至少有一半以上的黑色节点，则有
   红色节点个数：count(red)<=(n+1)/2；
   黑色节点个数：count(black)>=(n+1)/2;
假设x最短路径为(b1,b2.....bm)，根据性质(从该节点到后代的黑色节点相同),有
	路径节点数>黑色节点数，即m>=(n+1)/2
反正法证明，假设有n>2m，则有
	(n+1)/2>(2m+1)/2=m+1/2;
根据上面有 m>=(n+1)/2>m+1/2,出现矛盾，因此引理得证。

4. 内部节点最多为 2^(2bh+1)-1,最少为2^(bh+1)-1
	内部节点最多情形为，节点红黑交替，树的高度最大，值为2bh+1,因此节点数为2^(2bh+1)-1
	内部节点最少情形为，节点都是黑色，数的高度最小，值为bh,2^(bh+1)-1

##红黑树数据结构定义
```C
//红黑树颜色定义
typedef enum {
    RED,BLACK
} color_enum;

//红黑树的节点定义
typedef struct node {
    int data;	//节点数据内容
    struct node *left;	//左节点指针
    struct node *right;	//右节点指针
    struct node *parent;	//父节点指针
    color_enum color;		//颜色值
	int height;   //当前节点高度
} node_t;

//红黑树结构体定义
typedef struct {
    node_t *root;	//树的根节点
    int node_num;	//树的节点数量
} tree_t;

```

##树的旋转
代码实现如下，不好画图
```C
// 对x节点左旋
void left_rotate(tree_t *t,node_t* x)
{
 
   node_t *y = x->right;
 
   // 将y的左节点放到x的右指针上
	x->right=y->left;		
	if(y->left!=NULL){
		y->left->parent=x;
	}
	
	//y的父节点置为x的父节点，如果x父节点为空，说明x是树的根节点，如果不是，则将x的父节点保存的左/右指针指向y
	y->parent=x->parent;
    if(x->parent==NULL) {
        t->root=y;
    } else {
        if(x->parent->left==x) {
            x->parent->left=y;
        } else {
            x->parent->right=y;
        }
    }

    // y的左指针设置为x，x的父指针指向y
    y->left=x;
    x->parent=y;
}

// 对x节点右旋，与左旋相反
void right_rotate(tree_t *t,node_t *x)
{
    node_t *y=x->left;
	
	x->left=y->right;
	if(y->right!=NULL){
		y->right->parent=x;
	}
	y->parent=x->parent;
	if(x->parent==NULL) {
        t->root=y;
    } else {
        if(x->parent->left==x) {
            x->parent->left=y;
        } else {
            x->parent->right=y;
        }
    }

    y->right=x;
    x->parent=y;
}
```

##树的插入
```C
//红黑树插入
int insert_node(tree_t *t,int data)
{
	//创建一个节点
    node_t *node=(node_t *)malloc(sizeof(node_t));
    node->data=data;
	//所以插入的节点颜色默认为红色，由后面调整
    node->color=RED;
	node->left=node->right=NULL;
	node->parent=NULL;
    node->height=0;
    if(t==NULL) t=(tree_t *)malloc(sizeof(tree_t));
	//如果为空树，则直接设置返回
    if(t->root==NULL) {
        t->root=node;
        t->node_num=1;
        node->color=BLACK;
        return 0;
    }
    //在树中找到node适合的位置，确保树是有序的，parent保存node的父节点指针。
    node_t *nt=t->root,*parent;
    while(nt!=NULL) {
        parent=nt;
		if(data==nt->data) {
            printf("data:%d has already in the tree,return\n",data);
            return 0;
        } else if(data>nt->data) {
            // 右树中查找
            nt=nt->right;
        } else {
            //左树中查找
            nt=nt->left;
        }
    }
	// 插入树中
    if(data>parent->data) {
        parent->right=node;
    } else {
        parent->left=node;
    }
    node->parent=parent;
	
	//新节点插入树后，可能会引起不满足红黑树的特性，因此需调整树。    
	fix_after_insert(t,node);
    t->node_num++;
    return 0;
}
```
调整树代码如下：
```C
//求树颜色，如果节点为空，则节点为叶子节点，返回黑色
static inline color_enum color_of_node(node_t *x){
	return x==NULL?BLACK:x->color;
}

/*调整红黑树，分三种情况，以下case都是基于父节点为祖父节点的左节点来讨论，右节点则做相反操作即可
* case1. 当前节点的父节点为红色，父节点的兄弟节点为红色，此时将父节点与父节点的兄弟置为黑色，祖父节点置为红色，将当前节点指向祖父节点继续调整。
* case2. 当前节点的父节点为红色，父节点的兄弟节点为黑色，且当前节点为父节点的右节点，则对父节点进行左旋操作转换成case3
* case3. 当前节点的父节点为红色，父节点的兄弟节点为黑色，且当前节点为父节点的左节点，则将父节点置为黑色，祖父节点置为红色，对祖父节点进行右旋操作。 
*/
void fix_after_insert(tree_t *t,node_t *z)
{
    while(z!=NULL && color_of_node(z->parent)==RED) {
        //父节点为祖父节点的左节点
        if(z->parent==z->parent->parent->left) {
            //找到父节点的兄弟节点
            node_t *y=z->parent->parent->right;
            // case 1 start : 父节点为红色，父节点的兄弟节点为红色
            if(color_of_node(y)==RED) {
                z->parent->color=BLACK;     // 父节点置为红色
                z->parent->parent->color=RED;   //祖父节点置为红色
                y->color=BLACK;         //父节点的兄弟节点置为黑色
                z=z->parent->parent;    //当前节点置为祖父节点往上调整
            }
            //case 1 end
            else {
                // case 2 start : 父节点为红色，父节点的兄弟节点为黑色，z为父节点的右孩子
                if(z==z->parent->right) {
                    z=z->parent;
                    left_rotate(t,z);
                }
                //case 2 end ,转化成case 3
                z->parent->color=BLACK;
                z->parent->parent->color=RED;
                right_rotate(t,z->parent->parent);
            }
        } else {
			// 父节点为祖父节点的右节点，调整方式与左节点一样，反向操作
            if(z->parent==z->parent->parent->right) {
                node_t *y=z->parent->parent->left;
                if(color_of_node(y)==RED) {
                    z->parent->color=BLACK;
                    y->color=BLACK;
                    z->parent->parent->color=RED;
                    z=z->parent->parent;
                } else {
                    if(z->parent->left==z) {
                        z=z->parent;
                        right_rotate(t,z);
                    }
                    z->parent->color=BLACK;
                    z->parent->parent->color=RED;
                    left_rotate(t,z->parent->parent);
                }
            }
        }
    }
	//调整后，将根节点置黑
    t->root->color=BLACK;
}
```

##红黑树删除
