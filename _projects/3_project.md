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

Teaching robots contact-intesive tasks, such as peeling, scooping, and writing, from just demonstrations is another challenge i contributed to. In this work, using a reinforcement learning-based methodology without explicit reward engineering, we enable the robot to learn such contact intensive tasks. The robot system uses the modalities of touch (force and tactile feedback) and vision (videos of demonstrations). The experimental setup and the results are shown in Figure 2. The full paper is available <a href="https://ieeexplore.ieee.org/abstract/document/9561734">here</a>.  


<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.html path="assets/img/setup_main.png" title="The experimental Setup" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        {% include figure.html path="assets/img/writing_training_and_final.png" title="Writig Task" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The experimental setup and the writing task (training and final result) are shown in these images. 
</div>

