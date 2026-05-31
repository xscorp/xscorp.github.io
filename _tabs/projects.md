---
# the default layout is 'page'
icon: fas fa-code
order: 3
---

<style>
.card {
  border: 1px solid var(--border-color);
  background: var(--card-bg);
  transition: all 0.3s ease;
}

.card:hover {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  transform: translateY(-2px);
}

.card-img-top {
  background: var(--card-bg);
  padding: 8px;
  border-bottom: 1px solid var(--border-color);
  filter: brightness(var(--img-brightness, 1));
}

@media (prefers-color-scheme: dark) {
  .card-img-top {
    filter: brightness(0.9) contrast(1.1);
  }
}

@media (prefers-color-scheme: light) {
  .card-img-top {
    filter: brightness(1) contrast(1);
  }
}

.card-title {
  color: var(--text-color);
  font-weight: 600;
  margin-top: 12px;
}

.card-text {
  color: var(--text-muted-color);
  font-size: 0.95rem;
  line-height: 1.5;
}

.card .btn-primary {
  margin-top: 8px;
}

.row > .col-md-6 {
  display: flex;
}

.row > .col-md-6 .card {
  width: 100%;
}
</style>

## Featured Projects

<div class="row">
  <div class="col-md-6 mb-4">
    <div class="card h-100">
      <img src="https://opengraph.githubassets.com/default/xscorp/jsmug" class="card-img-top" alt="jsmug">
      <div class="card-body">
        <h5 class="card-title">jsmug</h5>
        <p class="card-text">The first and most evasive PoC of JSON Smuggling technique to smuggle arbitrary files through insignificant bytes.</p>
        <a href="https://github.com/xscorp/jsmug" class="btn btn-primary">View on GitHub</a>
      </div>
    </div>
  </div>

  <div class="col-md-6 mb-4">
    <div class="card h-100">
      <img src="https://opengraph.githubassets.com/default/Bevigil/BeVigil-OSINT-CLI" class="card-img-top" alt="bevigil-cli">
      <div class="card-body">
        <h5 class="card-title">bevigil-cli</h5>
        <p class="card-text">CLI tool and Python SDK to extract assets from a global database of 1M+ Android applications including URLs, subdomains, and more.</p>
        <a href="https://github.com/Bevigil/BeVigil-OSINT-CLI" class="btn btn-primary">View on GitHub</a>
      </div>
    </div>
  </div>

  <div class="col-md-6 mb-4">
    <div class="card h-100">
      <img src="https://opengraph.githubassets.com/default/xscorp/pingfisher" class="card-img-top" alt="pingfisher">
      <div class="card-body">
        <h5 class="card-title">pingfisher</h5>
        <p class="card-text">Tool to capture ICMPv4 and ICMPv6 ping requests and reply to assist during exploitation of blind vulnerabilities in CTFs.</p>
        <a href="https://github.com/xscorp/pingfisher" class="btn btn-primary">View on GitHub</a>
      </div>
    </div>
  </div>
</div>

---

## Other Contributions

I actively contribute to security research and open-source projects. Check out my [GitHub profile](https://github.com/xscorp) for more projects, detailed documentation, and contributions to the security community.
