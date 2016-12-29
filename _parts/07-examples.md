---
title: Примеры CRDT
---

Для описания CRDT описывают следующие методы (структуры):

- *payload* - локальная структура, в виде которой хранится состояние на узле
- *initial* - начальное состояние *payload*
- *query* - запрос на чтение состояния, не изменяющий его
- *update* - запросы на изменение данных
- *compare* - нахождение разницы между двумя состояниями
- *merge* - слияние с другим состоянием
- *atSource* - часть *update*, которая выполняется на текущем узле перед изменением состояния (*atSource* не меняет состояния)
- *downstream* - часть *update*, которая выполняется на данном узле и других узлах (репликах) для изменения состояния

Рассмотрим несколько примеров описания простых CRDT.

### CmRDT: Counter (счетчик)

{% highlight js %}
// payload
let i;

// initial
i = 0;

// query
let query = () => i;

// update increment
let update_increment = () => {
    // downstream
    i = i + 1;
};

// update decrement
let update_decrement = () => {
    // downstream
    i = i - 1;
};
{% endhighlight %}

### CvRDT: G-Counter (возврастающий счетчик)

{% highlight js %}
// payload
let p;

// initial
p = [0, 0, 0, 0, 0, ...]

// query
let query = () => {
    return p.reduce((a, b) => a + b, 0);
};

// update increment
let update_increment = () => {
    // downstream
    p[getNodeID()] += 1;
};

// compare
let compare = (x, y) => {
    for(let i = 0; i < x.length; i++) {
        if(x[i] > y[i]) {
            return false;
        }
    }
    return true;
};

// merge
let merge = (x, y) => {
    let tmp = [];
    for(let i = 0; i < x.length; i++) {
        tmp[i] = Math.max(x[i], y[i]);
    }
    return tmp;
};
{% endhighlight %}

### CvRDT: PN-Counter (возрастающий и убывающий счетчик)

{% highlight js %}
// payload
let p, n;

// initial
p = [0, 0, 0, 0, 0, ...]
n = [0, 0, 0, 0, 0, ...]

// query
let query = () => {
    let tmp = [];
    for(let i = 0; i < p.length; i++) {
        tmp[i] = p[i] - n[i];
    }
    return tmp.reduce((a, b) => a + b, 0);
};

// update increment
let update_increment = () => {
    // downstream
    p[getNodeID()] += 1;
};

// update decrement
let update_decrement = () => {
    // downstream
    n[getNodeID()] += 1;
};

// compare
let compare = (x, y) => {
    for(let i = 0; i < x.p.length; i++) {
        if(x.p[i] > y.p[i] || x.n[i] > y.n[i]) {
            return false;
        }
    }
    return true;
};

// merge
let merge = (x, y) => {
    let tmpp = [], tmpn = [];
    for(let i = 0; i < x.p.length; i++) {
        tmpp[i] = Math.max(x.p[i], y.p[i]);
        tmpn[i] = Math.max(x.n[i], y.n[i]);
    }
    return {p: tmpp, n: tmpn};
};
{% endhighlight %}

### CvRDT: G-Set (увеличивающийся список)

{% highlight js %}
// payload
let a;

// initial
a = new Set();

// query
let query = (el) => a.has(el);

// update add
let update_add = (el) => {
    // downstream
    a.add(el);
};

// compare
let compare = (x, y) => {
    return x.filter((el) => !y.has(el)).length == 0;
};

// merge
let merge = (x, y) => {
    var union = new Set(x);
    for (var el of y) {
        union.add(el);
    }
    return union;
};
{% endhighlight %}

### CvRDT: 2P-Set (увеличивающийся и уменьшающийся список)

{% highlight js %}
// payload
let a;
let r;

// initial
a = new Set();
r = new Set();

// query
let query = (el) => a.has(el) && !r.has(el);

// update add
let update_add = (el) => {
    // downstream
    a.add(el);
};

// update remove
let update_remove = (el) => {
    // atSource
    if(!a.has(el)) {
        return;
    }
    
    // downstream
    r.add(el);
};

// compare
let compare = (x, y) => {
    return x.a.filter((el) => !x.r.has(el))
        .filter((el) => !y.a.has(el) || y.r.has(el))
        .length == 0;
};

// merge
let merge = (x, y) => {
    var uniona = new Set(x.a);
    for (var el of y.a) {
        uniona.add(el);
    }
    var unionr = new Set(x.r);
    for (var el of y.r) {
        unionr.add(el);
    }
    return {a: uniona, r : unionr};
};
{% endhighlight %}