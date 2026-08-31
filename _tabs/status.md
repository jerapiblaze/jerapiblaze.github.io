---
# the default layout is 'page'
icon: fas fa-circle
title: System Status
order: 8
---

Belows are the status pages of different systems under my controls.

---

{% raw %}
<details>

<summary>ANSALab Systems</summary>

<a href="https://status-ansalab.j12tee.qzz.io/" target="_blank">Status page</a><br>
<a href="https://internal-status-ansalab.j12tee.qzz.io" target="_blank">Status page (self-hosted)</a><br>
<a href="https://grafana-ansalab.j12tee.qzz.io/" target="_blank">Grafana dashboard</a>

</details>
{% endraw %}

---

{% raw %}
<details>

<summary>LQT Compute Cluster</summary>

<a href="https://lqt-grafana-ansalab.j12tee.qzz.io/" target="_blank">Grafana dashboard</a>

</details>
{% endraw %}

---

{% raw %}
<details>

<summary>My own compute machine</summary>

<a href="https://tapi.j12tee.qzz.io/updates?key=teeport3000&lock=true" target="_blank">Status update page</a><br>
<a href="javascript:getUpdate()">Get Update Here</a><br>
<span id="update-content-box" style="white-space: pre-wrap; font-family: monospace;">Click the "Get Update Here" button to instantly get status update, or visit the status update page.</span>

</details>

<script>

function getUpdate(){
    let box = document.getElementById("update-content-box");
    box.innerText = "";
    fetch("https://tapi.j12tee.qzz.io/api/v1/updates/teeport3000").then(response => response.json()).then(data => {
      let box = document.getElementById("update-content-box");
      box.innerText = box.innerText + "\n" + data.value;
    })
}

</script>
{% endraw %}

---
