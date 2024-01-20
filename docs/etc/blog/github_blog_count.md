---
layout: post
title: github blog 방문자수 카운팅 배너 추가
description: add counting banner to github blog
date: 2024-01-20 14:10:00
last_modified_at : 2024-01-20 14:10:00
parent: Blog
grand_parent: Etc
has_children: false
tags: [github blog]
---

# goal..

- jekyll 테마로 구성한 github blog에 방문자 수 배너를 추가하는 것

# how to

- [https://otzslayer.github.io/etc/2023/08/13/add-view-counts-badge.html](https://otzslayer.github.io/etc/2023/08/13/add-view-counts-badge.html) 참조해서 [https://hits.seeyoufarm.com/](https://hits.seeyoufarm.com/)를 통해 배너를 생성하고
- default.html의 footer 부분 위쪽에 html 코드를 넣었습니다.
    
    ```html
    <footer class="site-footer">
      <div class="align-items-center w-100" style="text-align: center; margin-bottom: 1rem;">
        <img src="https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Ftnfhrnsss.github.io&count_bg=%23848ABA&title_bg=%23535050&icon=gnuicecat.svg&icon_color=%23E9DADA&title=view+count&edge_flat=false"/>
      </div>
    </footer>
    ```
    

# summary

- 페이지 메뉴를 클릭할 때마다, 페이지가 리로드되가보니 그때도 카운트가 됩니다. 순수 카운트를 얻기는 부족한 것 같습니다.
- 구글 서치엔진에서 취합하는 것으로 카운트를 얻고 싶지만, 취합이 실시간으로 되는 것 같지 않고, API도 없는 것 같네요