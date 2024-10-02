### 🔥 미션
***
#### 스터디 전의 쿼리 작성

**내가 진행 중, 진행 완료한 미션을 모아서 보는 쿼리 (페이징 포함)**
1. 진행 중인 미션 목록
    ```
    select member_name, restaurant_name, rm.points, introduction from member as m 
    inner join member_mission as mm on m.id = mm.member_id
    inner join (select m.*, restaurant_name from mission as m
        join (select restaurant.* from restaurant) as r 
        on m.restaurant_id = r.id where ongoing = 1) as rm
    on rm.id = mm.mission_id
    where m.id = 1
    order by m.created_at limit 5 offset 0;
    ```
2. 진행 완료한 미션 목록
    ```
    select member_name, restaurant_name, rm.points, introduction from member as m 
    inner join member_mission as mm on m.id = mm.member_id
    inner join (select restaurant_name, m.* from mission as m
        join (select restaurant.* from restaurant) as r 
        on m.restaurant_id = r.id where is_completed = 1) as rm
    on rm.id = mm.mission_id
    where m.id = 5
    order by m.created_at limit 5 offset 0;
    ```

**리뷰 작성하는 쿼리**
```
select nickname, restaurant_name, rating, content, image_url, rm.created_at, reply_content from restaurant as rest 
join (select m.nickname, ri.* from member as m 
	join (select i.image_url, i.created_at as image_created_at, rr.* from image as i 
		right join (select re.writer, re.created_at as reply_created_at, r.*, re.content as reply_content from review as r 
			left join reply as re 
            on re.review_id = r.id) as rr 
        on i.review_id = rr.id) as ri 
    on m.id = ri.member_id) as rm
on rest.id = rm.restaurant_id and rest.id = 4
order by created_at desc, reply_created_at desc, image_created_at desc limit 5 offset 0;
```

**홈 화면 쿼리 (현재 선택 된 지역에서 도전이 가능한 미션 목록, 페이징 포함)**
1. 현재 선택된 지역 및 진행 완료한 미션의 개수
    ```
    select location, count(*) as possible_missions from 
        (select m.id, ma.map_location as location from member as m 
        join map as ma on ma.id = m.residence_id) as mma
            inner join member_mission as mm on mma.id = mm.member_id
                inner join mission as mi on mm.mission_id = mi.id
    where mi.is_completed = 1 and mma.id = 5;
    ```
2. 현재 선택 된 지역에서 도전이 가능한 미션 목록
    ```
    select restaurant_name, kind, introduction, frm.points, frm.dday from member as mem
    inner join member_mission as mm on mm.mission_id = mem.id 
    inner join (select fr.kind, fr.restaurant_name, m.* from mission as m 
        join (select r.id, r.restaurant_name, fk.kind from food_kind as fk 
            inner join food_kind_restaurant as fkr on fkr.food_kind_id = fk.id
            inner join restaurant as r on fkr.restaurant_id = r.id) as fr on fr.id = m.restaurant_id
            where is_completed = 0 and ongoing = 0) as frm on frm.id = mm.mission_id
    where mem.id = 5
    order by frm.created_at limit 5 offset 0;
    ```
    
**마이 페이지 화면 쿼리**
```
select nickname, email, phone_number, points 
from member 
where id = 3
```