# 实验流程
## 准备工作
- 为了让edge collapse的过程更加清晰，我将其分为了几个阶段，每个阶段分别用一个model类的私有方法实现，以下为model.h中的声明，具体实现将在之后的报告中说明
```cpp
//remove correspond vertex edge and face in edge collapse
	void removeVertex(Halfedge* edge);
	void removeEdge(Halfedge* edge);
	void removeFace(Halfedge* edge);
```
- 为了正确地维护valid edge、vertex、face的数目，我对设置valid为false的过程进行了包装（后面发现实际上并不需要，在每次调用randomCollapse的时候会重新统计合法的数目）
```cpp
// model.h
// set edge face or vertex invalid
void setEdgeInvalid(Halfedge* target);
void setFaceInvalid(Face* target);
void setVertexInvalid(Vertex* target);

// model.cpp
// set invalid
void Model::setEdgeInvalid(Halfedge* target) {
	if (target) {
		//validEdges = target->valid ? validEdges - 1 : validEdges;
		target->valid = false;
	}
}

void Model::setFaceInvalid(Face* target) {
	if (target) {
		//validFaces = target->valid ? validFaces - 1 : validFaces;
		target->valid = false;
	}
}

void Model::setVertexInvalid(Vertex* target) {
	if (target) {
		//validVertices = target->valid ? validVertices - 1 : validVertices;
		target->valid = false;
	}
}
```
- 为了快速确定edge、vertex、face的index，我在frommesh函数中进行了修改，在添加半边、顶点、面的时候对id成员进行了维护
```CPP
// VERTEX
for (int i = 0; i < mvertices.size(); i++) {
		Vertex* v = new Vertex;
		v->position = mvertices[i]; 
		v->id = (int)vertices.size();
		this->vertices.push_back(v);
	}
// EDGE
for (int i = 0; i < 3; i++) {
			edge[i]->id = (int)halfedges.size();
			this->halfedges.push_back(edge[i]);
		}
// FACE
face->id = (int)faces.size();
		this->faces.push_back(face);
```
## opposite的计算
```cpp
//TODO: you are supposed to fill in the opposite attribute
	for (Halfedge* edges : this->halfedges) {
		// v0 是这条边的终点
		Vertex* v0 = edges->incident_vertex;
		// v2 是这条边的起点
		Vertex* v2 = edges->next->next->incident_vertex;
		edges->opposite = map[v2->id][v0->id];
	}
```
- 获取这个边的起点和终点下标，利用之前的map直接求得其反向边
- 对于不存在对边的边缘half-edge，在map中本身对应的元素就是nullptr，不影响结果
## removeVertex
主要思路为移除与这个halfedge有关的两个顶点，使用一个新的顶点代替二者。与原来那两个顶点有关的指针全部指向新的顶点。
```CPP
// remove two vertex of target edge and replace them with new one
void Model::removeVertex(Halfedge* edge) {
	Vertex* v1 = edge->incident_vertex;
	//get another vertex of cur halfedge
	Vertex* v2 = edge->next->next->incident_vertex;

	//set these two vertex invalid
	setVertexInvalid(v1);
	setVertexInvalid(v2);

	Vertex* vnew = new Vertex();
	//add a new vertex whose position is midpoint of above two
	vnew->position = (v1->position + v2->position) * glm::vec3(0.5, 0.5, 0.5);
	vnew->id = vertices.size();
	vnew->valid = true;
	vertices.push_back(vnew);
	//// redirect correspond edges
	//edge->incident_vertex = vnew;
	//edge->next->next->incident_vertex = vnew;
	//if (edge->opposite != nullptr) {
	//	edge->opposite->incident_vertex = vnew;
	//	edge->opposite->next->next->incident_vertex = vnew;
	//}
	for (auto edge : halfedges) {
		if (edge->incident_vertex == v1 || edge->incident_vertex == v2) {
			edge->incident_vertex = vnew;
		}
	}
	return;
}
```
## removeEdges
主要实现方法为移除这条边以及相邻的两条边，相邻两条边的opposite互为对方新的opposite即可。
```cpp
void Model::removeEdge(Halfedge* edge) {
	Halfedge* en = edge->next;
	Halfedge* enn = edge->next->next;
	setEdgeInvalid(edge);
	/*setEdgeInvalid(en->opposite);
	setEdgeInvalid(enn->opposite);*/
	setEdgeInvalid(en);
	setEdgeInvalid(enn);
	// there are chance that correspond edge dose not exist
	/*if (en) 
		en->opposite = enn;
	if (enn)
		enn->opposite = en;*/
	if (en->opposite)
		en->opposite->opposite = enn->opposite;
	if (enn->opposite)
		enn->opposite->opposite = en->opposite; 

	if (edge->opposite) {
		en = edge->opposite->next;
		enn = edge->opposite->next->next;
		setEdgeInvalid(edge->opposite);
		setEdgeInvalid(en);
		setEdgeInvalid(enn);
		if (en->opposite)
			en->opposite->opposite = enn->opposite;
		if (enn->opposite)
			enn->opposite->opposite = en->opposite;
	}
;
}
```
## removeFaces
简单地将本半边和opposite对应的face设置为invalid即可
```CPP
void Model::removeFace(Halfedge* edge) {
	setFaceInvalid(edge->incident_face);
	if (edge->opposite)
		setFaceInvalid(edge->opposite->incident_face);
}
```
## edgeCollapse
以上三个步骤的结合
```cpp
void Model::collapseEdge(int index) {
	//TODO: design a algorithm to collapse the face(edges) with the halfedge[index]
	Halfedge* cur = this->halfedges[index];

	removeVertex(cur);
	removeEdge(cur);
	removeFace(cur);
	//removeEdge(cur);
	//removeFace(cur);
	//TODO -- END
}

```
## 对randomCollapse的调整
将collpseEdge转移到For循环中，以便于一次删除大量的半边或面，当剩余的面较少时，将所有有效的edge全部删除，以防止出现bug
```CPP
// generate rand seed
	srand(time(0));
	dirty = true;
	/*while(true) {
		order_for_idx = rand() % halfedges.size();
		if (halfedges[order_for_idx]->valid) {
			break;
		}
	}*/
	for (int i = 0; i < 200; i++) {
		if (this->validFaces < 400) {
			for (int i = 0; i < halfedges.size(); i++) {
				if (halfedges[i]->valid)
					collapseEdge(i);
			}
			break;
		}

		srand(time(NULL));
		do {
			order_for_idx = rand() % halfedges.size();
		} while (!halfedges[order_for_idx]->valid);


		//TODO -- END

		collapseEdge(order_for_idx);

	}
```

# 问题
### face移除后直接消失
- 如下图所示：出现了面移除之后直接消失的情况，同时命令行中出现了对应的输出![[Pasted image 20221203203103.png]]![[Pasted image 20221203203042.png]]
- 怀疑可能是在removeVertex中重定位点的时候，有些指向原本invalid的halfedge没有被重定向到新加入的vertex上，因此我进行了如下的修改，每次遍历所有的半边，避免重定位不完全的情况出现
```cpp
//// redirect correspond edges
	//edge->incident_vertex = vnew;
	//edge->next->next->incident_vertex = vnew;
	//if (edge->opposite != nullptr) {
	//	edge->opposite->incident_vertex = vnew;
	//	edge->opposite->next->next->incident_vertex = vnew;
	//}
	for (auto edge : halfedges) {
		if (edge->incident_vertex == v1 || edge->incident_vertex == v2) {
			edge->incident_vertex = vnew;
		}
	}
```
### Faces和Edges数量变化不符合预期
- 如下图，当所有的三角形都被移除之后，还剩下大量的edge与faces，同时伴有异常的输出![[Pasted image 20221203211000.png]] ![[Pasted image 20221203211916.png]] 
- 发现是在removeEdge时，应该移除的是内侧的半边，之前实现错误，进行了如下的修改
```CPP
/*setEdgeInvalid(en->opposite);
	setEdgeInvalid(enn->opposite);*/
	setEdgeInvalid(en);
	setEdgeInvalid(enn);
	// there are chance that correspond edge dose not exist
	/*if (en) 
		en->opposite = enn;
	if (enn)
		enn->opposite = en;*/
	if (en->opposite)
		en->opposite->opposite = enn->opposite;
	if (enn->opposite)
		enn->opposite->opposite = en->opposite; 
```
### Faces数量异常
- 解决了以上问题之后，edge可以顺利被减少到0，但faces还是存在问题，并且还存在异常输出![[Pasted image 20221203214355.png]]
- 分析之后认为，由于removeEdges改变了边之间的opposite关系，因此必须在removeEdges之后再进行removeFace操作。否则可能出现由于opposite被改为了nullptr导致对应的face无法被及时移除的情况，进行了以下修改：
```CPP
removeVertex(cur);
removeEdge(cur);
removeFace(cur);
//removeEdge(cur);
//removeFace(cur);
```

# 最终效果
![[Pasted image 20221203215701.png]]
- 说明：为了快速删除完所有的边，我对于randomCollapse的框架进行了修改，一次选取200个edge进行remove（实际remove的数目可能大于200，被移除的halfedge如果存在oppsite，则会一次性移除6条半边，如果否则会移除3条半边）。为了防止最后一轮循环中合法的半边数目不足，因此我在face数量小于一定值时进行了特判，一次性删除剩下的所有半边。
```cpp
void Model::randomCollapse() {
	int order_for_idx = 0;
	//TODO: design a algorithm to fill the appropriate value into order_for_idx to call the function collapseEdge
	// you are supposed to ensure that each call to this function can reduce the number of face
	// generate rand seed
	srand(time(0));
	dirty = true;
	/*while(true) {
		order_for_idx = rand() % halfedges.size();
		if (halfedges[order_for_idx]->valid) {
			break;
		}
	}*/
	for (int i = 0; i < 200; i++) {
		if (this->validFaces < 400) {
			for (int i = 0; i < halfedges.size(); i++) {
				if (halfedges[i]->valid)
					collapseEdge(i);
			}
			break;
		}

		srand(time(NULL));
		do {
			order_for_idx = rand() % halfedges.size();
		} while (!halfedges[order_for_idx]->valid);


		//TODO -- END

		collapseEdge(order_for_idx);

	}

	//TODO -- END
	// collapse 200 edges once, move this call to the above loop
	//collapseEdge(order_for_idx);
	validFaces = statistic_cnt<Face*>(faces);
	validVertices = statistic_cnt<Vertex*>(vertices);
	validEdges = statistic_cnt<Halfedge*>(halfedges);
}

```
