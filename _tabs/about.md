---
# the default layout is 'page'
icon: fas fa-info-circle
order: 7
---

Minh-Thanh Nguyen
=================

PhD Candidate in Computer Science, School of Computer Science and Statistics, Trinity College Dublin.

I have knowledge about telecommunication networks (4G, 5G,...). With the vary programming skills from Python, JavaScripts, I can catch up with new working environments quickly. As a collaborative team player, I have experience in both independent and collaborative project work.

## Contact details

- <i class="fa-solid fa-envelope"></i> Email: <a target="_blank" href="mailto:nguyenm2@tcd.ie">nguyenm2 [at] tcd.ie</a>
- <i class="fa-solid fa-envelope"></i> Email: <a target="_blank" href="mailto:thanh.nm210802@sis.hust.edu.vn">thanh [dot] nm210802 [at] sis.hust.edu.vn</a>
- <i class="fa-solid fa-envelope"></i> Email: <a target="_blank" href="mailto:minhthanh.31oct2k3@gmail.com">minhthanh [dot] 31oct2k3 [at] gmail.com</a>
- <i class="fa-brands fa-google-scholar"></i> Google Scholar: <a target="_blank" href="https://scholar.google.com/citations?user=ie6HdJgAAAAJ">Minh-Thanh Nguyen</a>
- <i class="fa-brands fa-orcid"></i> ORCID: <a target="_blank" href="https://orcid.org/0009-0004-5291-029X">0009-0004-5291-029X</a>
- <i class="fa-brands fa-github"></i> GitHub: <a target="_blank" href="https://github.com/jerapiblaze">jerapiblaze</a>
- <i class="fa-brands fa-linkedin"></i> LinkedIn: <a target="_blank" href="https://www.linkedin.com/in/mthanh310/">mthanh310</a>

## Timezones

For your convinience, here are the clocks.

| Location | Timezone | Current time | Note |
| My location | <i id="main_tz">+0000</i>  | <a id="clock_main" target="_blank" href="/"></a> | <i id="main_diff"></i> |
| My hometown | <i id="sub_tz">+0700</i> | <a id="clock_sub" target="_blank" href="/"></a> | <i id="sub_diff"></i>
| Your location | <i id="you_tz"></i> | <a id="clock_you" target="_blank" href="/"></a> | |

  <script>
    function parseOffset(str) {
      const sign = str.startsWith('-') ? -1 : 1;
      const clean = str.replace('+','').replace('-','');
      const hours = parseInt(clean.slice(0,2), 10);
      const mins  = parseInt(clean.slice(2,4), 10);
      return sign * (hours * 60 + mins);
    }

    function getTimezoneOffsetString() {
      const offsetMinutes = new Date().getTimezoneOffset() * -1;

      const sign = offsetMinutes >= 0 ? "+" : "-";
      const abs = Math.abs(offsetMinutes);

      const hours = String(Math.floor(abs / 60)).padStart(2, "0");
      const mins  = String(abs % 60).padStart(2, "0");

      return sign + hours + mins;
    }

    function setYourTz(){
      const tzStr = getTimezoneOffsetString();
      document.getElementById("you_tz").textContent = tzStr;
    }
    setYourTz();

    function updateClock(tzInput, output) {
      const tzStr = document.getElementById(tzInput).textContent.trim();
      const offsetMinutes = parseOffset(tzStr);

      const now = new Date();
      const utc = now.getTime() + now.getTimezoneOffset() * 60000;
      const local = new Date(utc + offsetMinutes * 60000);

      document.getElementById(output).textContent =
        local.toLocaleString();
    }

    function updatePrimaryClock(){
      updateClock("main_tz", "clock_main");
    }
    function updateSecondaryClock(){
      updateClock("sub_tz", "clock_sub");
    }
    function updateYourClock(){
      updateClock("you_tz", "clock_you");
    }

    setInterval(updatePrimaryClock, 1000);
    updatePrimaryClock();
    setInterval(updateSecondaryClock, 1000);
    updateSecondaryClock();
    setInterval(updateYourClock, 1000);
    updateYourClock();

    function parseTZ(tz) {
      const sign = tz.startsWith('-') ? -1 : 1;
      const clean = tz.replace('+','').replace('-','');
      const hours = parseInt(clean.slice(0,2), 10);
      const mins  = parseInt(clean.slice(2,4), 10);
      return sign * (hours * 60 + mins);
    }

    function diffTimezone(tzA, tzB) {
      const offsetA = parseTZ(tzA);
      const offsetB = parseTZ(tzB);

      const diff = offsetA - offsetB;

      if (diff === 0) return "same as you";

      const ahead = diff > 0;
      const abs = Math.abs(diff);

      const hours = Math.floor(abs / 60);
      const mins  = abs % 60;

      let parts = [];
      if (hours > 0) parts.push(hours + " hour" + (hours === 1 ? "" : "s"));
      if (mins > 0) parts.push(mins + " minute" + (mins === 1 ? "" : "s"));

      return parts.join(" and ") + (ahead ? " ahead" : " behind");
    }
    function calcualteDiffs(){
      document.getElementById("main_diff").textContent = diffTimezone(
        document.getElementById("main_tz").textContent.trim(),
        document.getElementById("you_tz").textContent.trim(),
      );
      document.getElementById("sub_diff").textContent = diffTimezone(
        document.getElementById("sub_tz").textContent.trim(),
        document.getElementById("you_tz").textContent.trim(),
      )
    }
    calcualteDiffs();
  </script>

## CV

Download PDF: [English]({% link /assets/cvs/MinhThanh_Nguyen_CV_latest.pdf %})
