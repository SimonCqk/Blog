> 单源最短路径，不用说一定是经典的`dijkstra`算法了。`dijkstra`的原始版本做到了很好的解决问题，但是还有进一步优化的空间。

接下来展示通过“优先级队列（priority_queue）”优化时间复杂度的方法。
首先把图的类定义出来，我采用的是“class Edge”+"class Graph"的定义方案，将有向边和图的实现抽离，如有不妥，方请指正：
```cpp
template<typename Type>
class Edge
{
	template<typename Type>
	friend class Graph;
public:
	explicit Edge(const size_t& start, const size_t& end = 0, const Type& value = Type())
		:start_idx(start), end_idx(end), value(value), flag(Flags::unvisited) {}
	size_t getEnd() const { return end_idx; }
	size_t getValue() const { return value; }
	~Edge() = default;
private:
	size_t start_idx;
	size_t end_idx;
	Type value;
	Flags flag; // search for next_edge unvisied edge.
};
template<typename Type>
class Graph
{
	using Node = vector<Edge<Type>>;
	int INFTY = (1 << 20);
public:
	Graph() :
		num_edges(0) {}
	explicit Graph(const size_t& size);
	~Graph() = default;
    // some other functions.
    
	// return iterator points to the specific edge.
	decltype(auto) findEdge(const size_t& vertex_idx, const size_t& end) const;

	void addEdge(const size_t& vertex_idx, const size_t& end, const Type& value);
	void removeEdge(const size_t& vertex_idx, const size_t& end);

	void SingleSourcePath(); 
	void optmzed_SingleSourcePath();
private:
	void init();
	bool isConnected(const Node& node, const int& target);
	vector<Node> vertexs;
	size_t num_edges;
};
```
还需要一个`enum`结构来标记顶点或者有向边的状态：
```cpp
enum class Flags
{
	unvisited,
	be_visiting,
	visited,
	finish_visit
};
```
然后补充一下相关基础函数的操作：
```cpp
template<typename Type>
inline void Graph<Type>::init()
{
	for (auto& edges : vertexs)
		for (auto& edge : edges) {
			edge.flag = Flags::unvisited;
		}
}

template<typename Type>
inline decltype(auto) Graph<Type>::findEdge(const size_t & vertex_idx, const size_t & end) const
{
	assert(vertex_idx >= 0 && end >= 0);
	auto beg = vertexs[vertex_idx].begin(), ed = vertexs[vertex_idx].end();
	for (; beg != ed; ++beg) {
		if (beg->end_idx == end)
			return beg;
	}
	return ed;
}

template<typename Type>
inline void Graph<Type>::addEdge(const size_t & vertex_idx, const size_t & end, const Type & value)
{
	assert(vertex_idx >= 0 && end >= 0 && vertex_idx != end);
	if (vertex_idx > vertexs.size() || end > vertexs.size()) {
		vertexs.resize(max(vertex_idx, end) + 1);
	}
	vertexs[vertex_idx].push_back(Edge<Type>(vertex_idx, end, value));
	++num_edges;
}

template<typename Type>
inline void Graph<Type>::removeEdge(const size_t & vertex_idx, const size_t & end)
{
	assert(vertex_idx < vertexs.size() && end < vertexs.size());
	auto e = findEdge(vertex_idx, end);
	if (candi != vertexs[vertex_idx])
		vertexs[vertex_idx].erase(e);
}
```
接下来是正式的算法部分...图解的话...直接上书上的图片了：
![1](http://img.blog.csdn.net/20171014182624064?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY3FrMDEwMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![2](http://img.blog.csdn.net/20171014182640577?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY3FrMDEwMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
![eg](http://img.blog.csdn.net/20171014182651567?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY3FrMDEwMA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
实现如下：
```cpp
template<typename Type>
inline void Graph<Type>::SingleSourcePath()  // dijkstra's algorithm.
{
	init();
	int size = vertexs.size();
	vector<Flags> flags(size, Flags::unvisited);
	vector<int> ssp(size, INFTY);
	//initialize
	flags[0] = Flags::be_visiting;  // start from root.
	ssp[0] = 0;
	int idx, minv;
	while (true) {
		minv = INFTY;
		idx = -1;
		for (int i = 0; i < size; ++i) {
			if (minv > ssp[i] && flags[i] != Flags::visited) {
				minv = ssp[i];
				idx = i;
			}
		}
		if (idx == -1)  // all vertexs have benn added.
			break;
		flags[idx] = Flags::visited;
		//update ssp[].
		for (int i = 0; i < size; ++i) {
			if (flags[i] != Flags::visited && isConnected(vertexs[idx], i)) {
				int newv = findEdge(idx, i)->value;
				if (ssp[i] > ssp[idx]+newv) {
					ssp[i] = ssp[idx] + newv;
					flags[i] = Flags::be_visiting;
				}
			}
		}
	}
	for (int i = 0; i < size; ++i) {
		cout << i << " " << ssp[i] << endl;
	}
}
```
这个版本的算法中，每次更新ssp[]时，都会有一个forloop对所有顶点都扫描一遍，再加上外层的while主循环，一张V个顶点的图，其时间复杂度为O(|V|^2)，并不算很理想。倘若通过一个二叉堆来动态的维护这一个顶点的集合而不是每次暴力的遍历，那么其效率将会大大的提高。

优先级队列就是通过堆来实现的，用它来直接维护，操作会更方便。

下面给出优先级队列的版本：
```cpp
template<typename Type>
inline void Graph<Type>::optmzed_SingleSourcePath()
{
	// optimized single source path algorithm .
	// maintain by priority queue [ binary heap ]
	init();
	int size = vertexs.size();
	vector<Flags> flags(size, Flags::unvisited);
	vector<int> ssp(size, INFTY);
	// pair: first -> second corresponding to graph: start -> end
	std::priority_queue<std::pair<int, int>> pq; 
	//initialize
	flags[0] = Flags::be_visiting;  // start from root.
	ssp[0] = 0;
	pq.push(make_pair(0, 0));
	int idx;
	while (!pq.empty()) {
		pair<int, int> get = pq.top();
		pq.pop();
		idx = get.second;
		flags[idx] = Flags::visited;
		// get ssp[idx], if it's not the min, then skip
		if (ssp[idx] < get.first*(-1))
			continue;
		// update ssp[].
		int size_ = vertexs[idx].size();
		for (int i = 0; i < size_; ++i) {
			int v = vertexs[idx][i].end_idx;
			if (flags[v] == Flags::visited)
				continue;
			if (ssp[v] > ssp[idx] + vertexs[idx][i].value) {
				ssp[v] = ssp[idx] + vertexs[idx][i].value;
				pq.push(make_pair(ssp[v] * (-1), v));
				flags[v] = Flags::be_visiting;
			}
		}
	}
	for (int i = 0; i < size; ++i) {
		cout << i << " " << ssp[i] << endl;
	}
}
```

算法的时间复杂度下降到了O( (|V|+|E|)*log|V| )。