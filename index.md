---
layout: default-landing
title:  "Simple Sourcing"
hide_sidebar: true
permalink: /index.html
---

<div class="container">

    <!-- REMOVED <div class="jumbotron">
        <div class="container">
            <h1 class="text-center">A Java library for Event Sourcing</h1>
            <p class="text-center"><a href="docs_home.html" class="btn btn-primary">View Documentation</a></p>
        </div>
    </div>

    <div class="row">
        <div class="col-md-6 main-text">
            <h2 class="">What is Simple Sourcing?</h2>

              <p class="intro"><strong>Simple Sourcing</strong> is an API for building event sourcing systems where the data is stored in <a href="https://kafka.apache.org/">Kafka</a>. Event
              Sourced systems treat all the data as an immutable sequence of events. It is ideal for microservice architectures at scale to decouple data.</p>

        </div>
        <div class="col-md-6 text-center">
            <p hidden>TODO: Draw this ourselves:</p>
            <img style="width:50%;height:50%;" src="https://www.confluent.io/wp-content/uploads/2016/09/Event-sourced-based-architecture.jpeg">
        </div>
    </div> -->

    <div class="row callout">
        <div class="col-sm-4">
            <h1 class="text-center"><i class="get-started" aria-hidden="true"></i></h1>
            <h3 class="text-center">Easy to get started</h3>
            <p>Get started by linking the to the library from your code. Follow the <a href="quickstart.html">Quick Start</a> instructions.</p>
        </div>
        <div class="col-sm-4">
            <h1 class="text-center"><i class="api" aria-hidden="true"></i></h1>
            <h3 class="text-center">Small clean API</h3>
            <p>We have put a lot of effort into a very small clean API so it is quick to learn and start using. Read more about the <a href="programming_model.html">API here</a></p>
        </div>
        <div class="col-sm-4">
            <h1 class="text-center"><i class="read-only" aria-hidden="true"></i></h1>
            <h3 class="text-center">Simple read only views</h3>
            <p>Simple Sourcing supports many materialized views including MongoDB</p>
        </div>
    </div>
    <div class="row callout">
        <div class="col-sm-4">
            <h1 class="text-center"><i class="scalable" aria-hidden="true"></i></h1>
            <h3 class="text-center">Horizontally Scalable</h3>
            <p>You can run more than one instance of your application and behind the scenes,
                    the command handler and projectors will transparently share the work amongst all the running instances.</p>
        </div>
        <div class="col-sm-4">
            <h1 class="text-center"><i class="productive" aria-hidden="true"></i></h1>
            <h3 class="text-center">Developer Productivity</h3>
            <p>Spend your development time modeling your business domain and building up useful projections of your data.</p>
        </div>
        <div class="col-sm-4">
            <h1 class="text-center"><i class="reliable" aria-hidden="true"></i></h1>
            <h3 class="text-center">Consistent and Reliable</h3>
            <p>Simple Sourcing uses exactly-once delivery and optimistic locking to guarantee each event only
            appears once, in the order you expect.</p>
        </div>
    </div>

</div>
