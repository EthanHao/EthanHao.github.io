---
layout: default
---

 <div class="album text-muted">
  <div class="container">
   <div class="row">
	{% for project in site.opensource%}
	
	<div class="card">
	  <div class="card-block">
		<h4 class="card-title">{{ project.title }}</h4>
		<h6 class="card-subtitle text-muted">Support card subtitle</h6>
	  </div>
	  <img class="img-fluid" src="{{ project.thumbnail-path }}" alt="{{ project.title }}">
	  <div class="card-block">
		<p class="card-text">{{ project.short-description }}</p>
		<a href="{{ project.url }}" class="card-link">View It On Github</a>
	  </div>
	</div>
	{% endfor %}
 </div>
</div>
</div>

