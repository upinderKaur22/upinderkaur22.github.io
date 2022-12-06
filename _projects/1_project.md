---
layout: page
title: Closed-Loop Cyber-Physical System with Robots at the Edge for Precision Animal Agriculture
description: A closed-loop cyber-physical system with robots at the edge for improving animal welfare and sustainability in precision dairy farming 
img: /assets/img/cps_overview.jpg
importance: 1
category: work
---
Feeding a global population of 10 Billion by 2050 is a major challenge facing humanity! As nations get wealthier and urbanized, the consumption of animal-derived products, such as meat and dairy, is increasing. In the face of decreasing land and water availability as well as increasing concern for the ethical treatment of animals, boosting productivity and sustainability of agriculture is the problem I undertake in this work. 

Research indicates that boosting animal welfare can increase the productivity of current livestock operations. Precision animal agriculture or precision livestock farming (PLF), as a movement, encourages returning to the "per animal approach", wherein care is customized according to the needs of individuals rather than based on aggregates. Since commercial farms today have more than 10,000 animals, it is impossible to care for individuals manually. 

Cyber-physical systems have proven their mettle in crop agriculture by delivering increased productivity, real-time care, and optimized use of resources. However, animal agriculture is a much greater challenge. Each individual is a valuable, sentient being with autonomy of movement and behvior unlike crops. While it takes weeks to months for changes to precipitate in crops, animal heath can change in a matter of hours. Animals not only have individuality but are social beings too. Therefore, the dynamics of the herd plays a major role in their overall wellbeing. Therefore, I created a novel closed-loop cyber-physical systems (CPS) with robots at the edge that is generalizable across the various domains of animal agricluture, ensuring individualized care with real-time monitoring. 

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/cps_overview.jpg" title="Closed-Loop Cyber-Physical System with robots at the edge for Precision Animal Agriculture " class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The overview of the architecture for the closed-loop CPS with in-vivo robots for precision dairy farming. 
</div>

The edge comprises the in-vivo robot that monitors and maps the rumen (the stomach of the animal). Even today little is known about the inner-workings of the highly dynamic and stratified environment. The in-vivo robot hosts valuable sensors that collect information about key biomarkers, such as pH, temperature, and ammonia. The biomarker activity also changes with time and with the cycles of heat and estrus. Therefore, the application needs an adaptive sensing node. For this, I built <ib>CASPER: A Criticality-Aware Self-Powered Wireless in-vivo Sensing Edge for Precision Animal Agriculture</ib> which won the Best Paper Award at the AgSys Workshop of SenSys, 2022.
<div class="row">
    <div class="col-sm">
        {% include figure.html path="assets/img/SenSYS_sys_overview.jpg" title="CASPER" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm">
        {% include figure.html path="assets/img/SenSYS_cow_sensor.jpg" title="CASPER" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    CASPER (a)Architecture and (b) Deployment. 
</div>

Getting the data out from inside the animal needed a robust wireless communication link from inside the body to the outside. Therefore, we conducted the seminal study for identifying the optimal freqeuncy for RF-based wireless communication. The setup is shown in the figure below. The full paper is available <a href="https://ieeexplore.ieee.org/abstract/document/9629743"> here</a>. 
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/Introduction_Figure.jpg" title="layout for experiments on RF comm" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The characterization of the channel response for in-body to out-of-body communication at 400MHz included RSSI measurments around the animal. These RF measurements were combined with a 3D model of the physiology of the animal to generate a first-of-its-kind HFSS model of the animal's body. 
</div>

Book Chapter
<a href="https://www.academia.edu/download/67422528/Improving_Animal_Welfare_A_Practical_Approach_3rd_Edition_Booksvets.blogspot.com_.pdf#page=571"> Future of Animal Welfare: Technological Innovations for Individualized Animal Care</a>.

Invited Reviews 
<a href="https://www.sciencedirect.com/science/article/pii/S0022030222003502">Invited review: Sensor technologies for real-time monitoring of the rumen environment</a>.
 

