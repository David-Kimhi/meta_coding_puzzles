<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Boss Fight</title>
<!-- Load MathJax for nicer math rendering -->
<script src="https://polyfill.io/v3/polyfill.min.js?features=es6"></script>
<script src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-chtml.js"></script>
<style>
  body { font-family: sans-serif; margin: 2em; line-height: 1.5; }
  code { background: #f8f8f8; padding: 2px 4px; }
  pre { background: #f8f8f8; padding: 1em; }
</style>
</head>
<body>
<h1>Two Warriors vs. Boss Problem</h1>
<p>We have <em>N</em> warriors, each with:</p>
<ul>
<li><strong>H<sub>i</sub></strong>: Health</li>
<li><strong>D<sub>i</sub></strong>: Damage Per Second (DPS)</li>
</ul>
<p>A boss with DPS <strong>B</strong> fights two warriors in sequence:</p>
<ol>
<li>The “front‐line” warrior \(i\) fights together with warrior \(j\) until the death of \(i\).</li>
<li>The “back‐up” warrior \(j\) fights until death.</li>
</ol>
<p>We want the <em>maximum total damage</em> two chosen warriors can deal to the boss, over any pair \((i,j)\).</p>
<p>Formally, if \(i\) goes first and \(j\) second, the damage is:
\[
(H_i \times D_i) + ((H_i + H_j)\times D_j) \Big/ B.
\]
    <i>Note:</i> Since the first warrior must be one of the <em>N</em> warriors, <b>we don't need to calculate the reverse case for each warrior</b>, i.e., the case <code>(j, i)</code>.</p>
<hr>
<h2>Why a Data Structure?</h2>
<p>A naive approach checks all \((i,j)\) in \(O(N^2)\). For large \(N\), this is too slow.
    <br>We notice the expression
\(
H_i D_i + (H_i + H_j)D_j
\)
can be reorganized into a “linear function” part, allowing a <em>Li Chao Tree</em> (or Convex Hull Trick) to handle queries in about
    \(
        O(N\log(N\\))
    \).</p>
<hr>
<h2>The Li Chao Tree Approach</h2>
<h3>1. Linear Functions</h3>
<p>Each warrior \(j\) corresponds to a line
\[
y_j(x) \;=\; D_j\cdot x \;+\; (H_j \cdot D_j),
\]
where \(x\) will be \(H_i\). Then \((H_i+H_j)D_j = D_j\cdot H_i + (H_j D_j)\).
    <br>If we fix \(i\), the constant part is \(H_i D_i\), and the rest depends on \(j\): \(D_j\cdot H_i + H_j D_j\).
    <br>A Li Chao Tree can quickly find which \(j\) yields the maximum \(D_j\cdot x + (H_jD_j)\) at \(x=H_i\).</p>
<h3>2. Excluding the Same Index</h3>
<p>Because we can not use the same warrior as the first and the second, we must exclude the case \(i=j\).
    <br>That means, that when we're querying the tree to find the maximum value between the lines, we have to ignore the value of the \(i\)th line, and instead consider the next best line.
    <br>In our code we did that by storing a secondary line for the fallback if the best line has the same index as the one who called the query function initially.</p>
<hr>
<h2>Explanation of the Code</h2>
<pre><code>class Line:
    def __init__(self, m, b, index, a_node=True):
        self.m = m
        self.b = b
        self.index = index
        if a_node:
            self.second_best_right = Line(0,0,index,False)
            self.second_best_left  = Line(0,0,index,False)
    def __call__(self,x):
        return self.m*x + self.b
</code></pre>
<p><strong>Key Points</strong>:</p>
<ul>
<li><code>Line(m,b,index)</code> represents one warrior’s line \(y=m\,x+b\).</li>
<li><code>second_best_left</code> and <code>second_best_right</code> are fallback lines to exclude the same index.</li>
</ul>
<pre><code>def insert(l,r,segment,idx=0):
    if (not segment.m and not segment.b) or idx>=N: return
    if l+1==r:
        if segment(l)>a[idx](l): a[idx]=segment
        return
    mid=(l+r)//2
    if a[idx].m>segment.m: a[idx],segment=segment,a[idx]
    if a[idx](mid) < segment(mid):
        a[idx],segment=segment,a[idx]
        insert(l,mid,segment,idx*2+1)
    else:
        insert(mid,r,segment,idx*2+2)
</code></pre>
<p><strong>Key Points</strong>:</p>
<ul>
<li>We store the better line at <code>a[idx]</code> and recurse the “loser” into the child subrange.</li>
</ul>
<pre><code>def query(l,r,x,query_index,idx=0):
    if idx>=len(a): return 0
    if l+1==r: return a[idx](x)
    mid=(l+r)//2
    if x < mid:
        if query_index!=a[idx].index:
            return max(a[idx](x),query(l,mid,x,query_index,idx*2+1))
        else:
            return max(a[idx].second_best_right(x),
                       a[idx].second_best_left(x),
                       query(l,mid,x,query_index,idx*2+1))
    else:
        if query_index!=a[idx].index:
            return max(a[idx](x),query(mid,r,x,query_index,idx*2+2))
        else:
            return max(a[idx].second_best_right(x),
                       a[idx].second_best_left(x),
                       query(mid,r,x,query_index,idx*2+2))
</code></pre>
<p><strong>Key Points</strong>:</p>
<ul>
<li>When querying at <em>x</em>, if <code>a[idx].index</code> is not the same as <em>query_index</em>, use <code>a[idx](x)</code>. Otherwise, use <code>second_best_*</code> lines.</li>
</ul>
<pre><code># Build the Li Chao tree
a=[Line(0,0,i) for i in range(N)]
min_range=min(H)
max_range=max(H)
for j in range(N):
    slope_j=D[j]
    intercept_j=H[j]*D[j]
    insert(min_range,max_range,Line(slope_j,intercept_j,j))
for i in range(N):
    l_child=i*2+1
    r_child=i*2+2
    if l_child < N: a[i].second_best_left=a[l_child]
    if r_child < N: a[i].second_best_right=a[r_child]
answer=max(D[i]*H[i]+query(min_range,max_range,H[i],i) for i in range(N))/B
</code></pre>
<p><strong>Key Points</strong>:</p>
<ul>
<li>After insertion, each node <code>a[idx]</code> is the best line for that segment.</li>
<li>We link <code>second_best_left</code> and <code>second_best_right</code> to children for excluding the same index.</li>
<li>We then query for each warrior \(i\), summing its front-line part \((H_iD_i)\) with the best back-line from the tree, ensuring \(j\neq i\).</li>
<li>Finally, we divide by <em>B</em>.</li>
</ul>
<hr>
<h2>Putting It All Together</h2>
<p>We effectively store lines \(\ell_j(x)=D_j x + H_jD_j\) for all \(j\). Then for each \(i\), we query \(\max_{j\neq i}\ell_j(H_i)\) and add \(H_iD_i\). The best result is returned after dividing by <em>B</em>.</p>
<p>This approach reduces the naive \(O(N^2)\) process to about \(O(N\log(\mathrm{range}))\), making it feasible for large \(N\).</p>
</body>
</html>