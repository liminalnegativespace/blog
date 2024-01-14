+++
title = "给博客添加heatmap"
summary = " "
image = "images/header.png"
categories = [
    "故纸堆",
]
tags = [
    "技术",
    "教程",
]
date = "2024-01-14"
menu = "main"
+++

{{< hint info>}}
感谢[椒盐豆豉的教程](https://blog.douchi.space/hugo-blog-heatmap)
{{< /hint >}}

因为我想要在鼠标移动到某个日期的时候显示所有在这一天发布的文章链接（虽然目前为止还没有在同一天发布超过一篇文章···），于是对代码进行了一些改动。

要实现这个功能就需要将某个特定日期下发布的所有文章放进一个array里，与当日的总字数一起储存在由日期作为key的map里面：

```tpl
var postsByDate = new Map();

{{ range ((where .Site.RegularPages "Type" "posts"))  }}
    var date = {{ .Date.Format "2006-01-02" }};
    var postObj = new Map();
    postObj.set("title", {{ .Title }});
    postObj.set("link", {{ .RelPermalink }});
    var wordCount = {{ .WordCount }};

    var data = postsByDate.get(date);
    if (data === undefined) {
        data = new Map();
        data.set("posts", []);
        data.set("totalWordCount", 0);
    }
    var posts = data.get("posts");
    posts.push(postObj);
    var totalWordCount = data.get("totalWordCount");
    totalWordCount += wordCount;
    data.set("totalWordCount", totalWordCount);
    postsByDate.set(date, data);
{{- end -}}
```
上面的代码执行完以后`postsByDate`的数据结构会变成类似下面这个样子：

```tpl
{
    '2022-10-13': {
        'posts': [
            {'title': '一篇文章', 'link': 'posts/some-post'},
            {'title': '另一篇文章', 'link': 'posts/another-post'},
        ],
        'totalWordCount': 404,
    },
    '2022-11-24': {
        'posts': [
            {'title': '又一篇文章', 'link': 'posts/one-more-post'},
        ],
        'totalWordCount': 401,
    },
}
```

接下来需要改动`option`中`tooltip`上的`formatter`这个attribute来使其显示当日的所有文章链接。前面已经把文章array储存在`postsByDate`里了，这里只需要从传入的参数里提取出对应的`date`再以此为key从`postsByDate`里获取文章信息（关于参数的结构可以参见<a href="https://echarts.apache.org/zh/option.html#tooltip.formatter" target="_blank">官方文档</a>）。这里可以直接返回构建好的html string：
```tpl
function(params) {
    const date = params.data[0];
    const posts = postsByDate.get(date).get("posts");
    var content = `${date}`;
    for (const [i, post] of posts.entries()) {
        content += "<br>";
        var link = post.get("link");
        var title = post.get("title");
        content += `<a href="${link}" target="_blank">${title}</a>`
    }
    return content;
}
```

{{< expand "完整代码">}}
`layouts/partials/heatmap.html`
```tpl
<div 
    id="heatmap" 
    style="
    display: block;
    height: 150px;
    width: 75%;
    padding: 2px;
    text-align: center;"
></div>
<script src="https://cdn.jsdelivr.net/npm/echarts@5.3.0/dist/echarts.min.js"></script>
<script type="text/javascript">
    var chartDom = document.getElementById('heatmap');
    var myChart = echarts.init(chartDom);
    window.onresize = function() {
        myChart.resize();
    };
    var option;

    // get date range by number of months
    function getDateRange(months){            
        var startDate = new Date();
        var mill = startDate.setMonth((startDate.getMonth() - months));
        var endDate = +new Date();
        startDate = +new Date(mill);

        endDate = echarts.format.formatTime('yyyy-MM-dd', endDate);
        startDate = echarts.format.formatTime('yyyy-MM-dd', startDate);

        var dateRange = [];
        dateRange.push([
            startDate,
            endDate
        ]);
        return dateRange
    };

    // get number of months by window size
    function getMonthCount(){
        const windowWidth = window.innerWidth;
        if (windowWidth >= 600) {
            return 12;
        }
        if (windowWidth >= 400) {
            return 9;
        }
        return 6;
    }

    var postsByDate = new Map();
    {{ range ((where .Site.RegularPages "Type" "posts"))  }}

        var date = {{ .Date.Format "2006-01-02" }};
        var postObj = new Map();
        postObj.set("title", {{ .Title }});
        postObj.set("link", {{ .RelPermalink }});
        var wordCount = {{ .WordCount }};

        var data = postsByDate.get(date);
        if (data === undefined) {
            data = new Map();
            data.set("posts", []);
            data.set("totalWordCount", 0);
        }
        var posts = data.get("posts");
        posts.push(postObj);
        var totalWordCount = data.get("totalWordCount");
        totalWordCount += wordCount;
        data.set("totalWordCount", totalWordCount);
        postsByDate.set(date, data);
    {{- end -}}

    var heatmapData = [];
    for (const [date, data] of postsByDate.entries()) {
        heatmapData.push([date, data.get("totalWordCount")]);
    }

    option = {
    title: {
        show: true,
        top: 0,
        left: 'center',
        text: '文章日历',
    },
    legend: {
        show: false,
    },
    visualMap: {
        show: false,
        min: 0,
        max: 10000,
        type: 'piecewise',
        showLable: false,
        orient: 'horizontal',
        left: 'center',
        top: 0,
        itemGap: 10,
        inRange: {
            color: ['#c4e8ff', '#47b4fc', '#0269ad'],
        },
    },
    calendar: {
        top: 50,
        left: 30,
        right: 30,
        cellSize: ['auto', 'auto'],
        range: getDateRange(getMonthCount()),
        itemStyle: {
            color: '#fff',
            borderWidth: 0.5,
            borderColor: '#eee',
        },
        yearLabel: { 
            show: false,
        },
        dayLabel: {
            align: 'center',
            nameMap: 'ZH',
        },
        monthLabel: {
            nameMap: 'EN',
        },
        splitLine: {
            show: false,
        },
    },
    tooltip: {
        hideDelay: 1000,
        enterable: true,
        formatter: function(params) {
            const date = params.data[0];
            const posts = postsByDate.get(date).get("posts");
            var content = `${date}`;
            for (const [i, post] of posts.entries()) {
                content += "<br>";
                var link = post.get("link");
                var title = post.get("title");
                content += `<a href="${link}" target="_blank">${title}</a>`
            }
            return content;
        }

    },
    series: {
        type: 'heatmap',
        coordinateSystem: 'calendar',
        data: heatmapData
    }
};
option && myChart.setOption(option);
</script>
```
`在其他页面中插入heatmap` 
```tpl
{{ partial "heatmap" . }}
```
{{< /expand >}}
