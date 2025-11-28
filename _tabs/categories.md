---
layout: categories
icon: fas fa-stream
order: 1
---

<script>
  document.addEventListener("DOMContentLoaded", function() {
    // 1. 접혀있는 모든 폴더 내용(.collapse)을 찾아서 펼침(.show) 처리
    var collapses = document.querySelectorAll('.collapse');
    collapses.forEach(function(collapse) {
      collapse.classList.add('show');
    });

    // 2. 화살표 아이콘 상태를 '펼쳐짐'으로 변경 (화살표 방향 맞추기)
    var triggers = document.querySelectorAll('[data-bs-toggle="collapse"]');
    triggers.forEach(function(trigger) {
      trigger.setAttribute('aria-expanded', 'true');
    });
  });
</script>