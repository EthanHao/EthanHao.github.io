---
layout: default
---

 <div class="album text-muted">
 <div class="row">
	{% for project in site.portfolio%}
	<div class="col-sm-6">
	<div class="card">
	  <div class="card-block">
		<h4 class="card-title">{{ project.title }}</h4>
		<h6 class="card-subtitle text-muted">{{ project.traits }}</h6>
	  </div>
	  <img class="img-fluid" src="{{ project.thumbnail-path }}" alt="{{ project.title }}">
	  <div class="card-block">
		<p class="card-text">{{ project.short-description }}</p>
		<a href="{{ project.Address }}" class="card-link">View More</a>
	  </div>
	</div>
	</div>
	{% endfor %}
</div>
</div>


 