---
layout: page
title: Learning Contact Intensive Tasks Using Multimodal Preception and Reinforcement Learning
description: Learning Multimodal Skills using reinforcement learning
img: assets/img/overview_ral.jpg
importance: 2
category: work
---
Humans effectively manipulate deformable objects by leveraging their multimodal perception for everydays tasks such opening bags or candy and finding keys in pockets. Teaching such tasks to robots is a non-trivial problem. 

For enabling robots to learn multimodal manipulation, especillay for deformable objects, we built a human-like exploration and purposeful manipulation framework. The framework, as shown in Figure 1, allows robots to learn and adapt to a class of deformable objects, autonomously. Further, it enables robots to purposefully manipulate the objects, by levergaing the knowledge base created during the exploration stage, to complete a task. Please refer our paper for more details. 


<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/overview_ral.jpg" title="The Multimodal Perception Framework" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The multimodal perception framework learns a Ziplock bags during exploratory manipulation by building a knowledge base of multimodal represnetations. The knowledge base is then used to comleplte the task of closing the bag during purposeful manipulation.
</div>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/5.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    This image can also have a caption. It's like magic.
</div>

You can also put regular text between your rows of images.
Say you wanted to write a little bit about your project before you posted the rest of the images.
You describe how you toiled, sweated, *bled* for your project, and then... you reveal it's glory in the next row of images.


<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.html path="assets/img/6.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.html path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    You can also have artistically styled 2/3 + 1/3 images, like these.
</div>


The code is simple.
Just wrap your images with `<div class="col-sm">` and place them inside `<div class="row">` (read more about the <a href="https://getbootstrap.com/docs/4.4/layout/grid/">Bootstrap Grid</a> system).
To make images responsive, add `img-fluid` class to each; for rounded corners and shadows use `rounded` and `z-depth-1` classes.
Here's the code for the last row of images above:

{% raw %}
```html
<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.html path="assets/img/6.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.html path="assets/img/11.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
```
{% endraw %}
