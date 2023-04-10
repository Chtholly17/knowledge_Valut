1. Write a pseudo code to implement a mesh query， i.e. given a vertex v0, find all its surrounding vertices, by using the half-edge data structure (according to the following definition ).
	实现mesh query功能的代码如下，原理为首先通过incident edge找到一条半边，将这个半边所在的三角形中v0对应半边指向的顶点加入结果中，周边的三角形除了v0之外还存在其他的公共顶点，为防止重复，每个三角形只需添加一个顶点到结果中即可。之后通过oppsite找到相邻且公共顶点为v0的下一个三角形，重复以上操作直到访问了顶点中包含v0的所有三角形。此算法要求在流形的情况下才可以得到正确的结果。
```cpp
/*
* @brief
* implementation of mesh query using given half-edge structure
* @param
* v0: the given vertex
* @return
* res: all surrounding vertex of v0
* @attention
* this algorithm only works properly with manifold situation
*/
vector<Vertex*> meshQuery(Vertex &v0){
	Halfedge * first = v0.incident_halfedge;
	//first half edge of current triangle
	Halfedge * cur = first;
	vector<Vertex*> res;
	while (true)
	{
	// get surrounding vertex in cur triangle
	cur = cur->next;
	res.push_back(cur->incident_vertex);
	// goto next triangle
	cur = cur->next->opposite;
	if(cur == first)
		break;
	}
	return res;
}
```
2. Write a pseudo code to draw a Bezier curve give four points P0 , P1 , P2 , P3
	以下为使用de_Casteljau算法迭代绘制贝塞尔曲线的伪代码，其中segNum为绘制的贝塞尔曲线上点的数目，segNum越大，绘制的曲线越平滑。最终曲线上所有的点存放在list Vertex中
```python
segNum = 100
def de_Casteljau(p0,p1,p2,p3):
	vertex = []
	for i in range(0,segNum):
		alpha = 1.0 / segNum * i
		beta = 1.0 - alpha
		cur = pow(alpha,3)* p0 +\
			 3 * pow(alpha,2) * beta * p1 +\
			 3 * pow(beta,2) * alpha * p2 +\
			 pow(beta,3)* p3
	vertex.append(cur)
	return
```
3. Write a pseudo code to implement the raytracing with BVH optimization (one ray per pixel and no sampling for diffuse).
```python
BVHNode bvh_root
'''
	@brief
		generate all object which is intersected with cur ray
'''
def object_generation(node:BVHNode,r:ray,NodeList:list):
	if node.isNULL():
		return
	if r intersect node 
		if BVNNode.isleaf():
			NodeList.append(node)
		else:
			object_generation(node.rchild,r,NodeList)
			object_generation(node.lchild,r,NodeList)

'''
	@brief
		calculate ray tracing recursively
'''
def ray_tracing(r:ray):
	NodeList = []
	object_generation(bvn.root,r,NodeList)
	# sort all Nodes by distance
	NodeList.sort()
	while(Ture):
		if NodeList.empty():
			return
		cur = NodeList[0]
		# generating reflect light, transmit light, shadow light
		if intersect(cur,r):
			rr = generate_reflect(cur,r)
			tr = generate_transmit(cur,r)
			sr = generate_shadow(cur,r)
			# calculate rr and tr recursived
			ray_tracing(rr)
			ray_tracing(tr)
			# handling shadow light
			handleShadowLight(sr,r)
		else:
			NodeList = NodeList[1::]

'''
	@brief:
		calculate if shadow light hit other object
'''
def handleShadowLight(sr:ray,r:ray):
	# get node concluding source of ray r
	sourceObj = r.source()
	NodeList = []
	object_generation(bvn.root,sr,NodeList)
	# sr intersect with no object except source object of r
	if NodeList.size() != 1 or NodeList[0] != sourceObj:
		setShadow(False)
	else 
		setShadow(True)
	draw()

def main():
	bvh_root = bvh_generating()
	ray_list = []
	# generating all rays, using monter carlo is an option
	generate_ray(ray_list)
	for ray in ray_list:
		ray_tracing(ray)

if __name__ = 'main':
	main()
```

