
[Isobar公司的前端代码规范和最佳实践](http://blog.jobbole.com/79075/)

对前端开发的各个方面做了详尽描述，还提及到很多有用的工具和资源。

对于所用样式只出现一次的元素，给它设定一个id属性。这个属性只会应用于该元素，不会用到其他地方。Class属性则可以用到多个具有相同样式属性的元素上。具有相同外观和表现的元素可以具有相同的class名。

    <ul id="categories">
        <li class="item">Category 1</li>
        <li class="item">Category 2</li>
        <li class="item">Category 3</li>
    </ul>

良好的web设计和开发的一个重要部分就是SEO。