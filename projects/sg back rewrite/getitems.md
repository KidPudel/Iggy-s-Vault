~~4~~ 3 queries


# params
- deptid
- addressid
# Summary info
- get dept info (deptid, deptstatus, name etc)
- get items

---

# 1 get dept id from user address (yandex)
If parameters not set deptid or deptid empty or address > 0
 - then if **user exist, we find out the deptid** by giving user's address id (and user id) from yandex houses
```
$stm = DB::$db->prepare("select dept.id as deptid,dept.status as deptstatus,
			(select jsonb_agg(to_jsonb( files) - 'id' - 'path') from files
    		join file_links fl on fl.fileid = files.id and fl.active =1 and  fl.recordid = dept.id and fl.linktypeid in(7,8) group by fl.recordid ) as deptimg
     		,dept.timedelay,dept.name
			from address_link link 
    		join addresstable a on link.addressid = a.id 
    		join yandex_houses house on house.id = a.ora_id
		    join yandex_houses_in_area yhia on yhia.house_id = house.id
			join dept_areas area on yhia.main_area_id = area.ora_id
			join departments dept on dept.id = area.dept_id
			where link.recid = :userid and (:addressid = -1 or link.addressid = :addressid) and link.appruved  = 3 order by link.id desc
     		limit 1");
```
```php
$res = $stm->execute([
				'userid'=>Auth::$auth->user->id,'addressid'=>$params->addressid]);
```
-  if dept is not empty and dept.deptid, assign params.deptid to the dept.deptid
- else set params deptid to the 1257 (бутырка)


Later if params dept id is still empty, through an error
```php
	die(json_encode(['status'=>'error','errorcode'=>'700','msg'=>'parameters mismatch']));
```

# 2 get dept id from not registered user (if dept is empty)
```
$params->deptid = $params->deptid == '1114' ? 1257 : $params->deptid;
	$stm = DB::$db->prepare( "select dept.id as deptid,dept.status as deptstatus,
		(select jsonb_agg(to_jsonb( files) - 'id' - 'path') from files
    		join file_links fl on fl.fileid = files.id and fl.active =1 and  fl.recordid = dept.id and fl.linktypeid in(7,8) group by fl.recordid ) as deptimg
     ,dept.timedelay
	 from departments dept
     where  dept.id = greatest (:deptid,1257) limit 1");
```
```php
$res = $stm->execute([
		'deptid' => $params->deptid]);
```
if dept exists and dept.deptstatus != 1
```php
	die(json_encode(['status'=>'error','msg'=>'К сожалению, новые заказы пока не принимаем.','errorcode'=>'100','error'=>'stopped department','params'=>['deptid'=>$dept->deptid,'deptstatus'=>$dept->deptstatus,'deptimg'=> json_decode($dept->deptimg) ?? 0]]));
```

# 3 get the items and rest of info
```sql
select items.id, items.id as itemid, items.name as name,
	'0'::integer as profit, cat.name as catname, items.category_id,
	coalesce (descr.text, items.name) as description,
	concat('МЕНЮ / ', cat.name, ' / ', items.name) as breadcrumbs,
	(select jsonb_agg(to_jsonb( files) - 'id' - 'path') from files
	join file_links fl on fl.fileid = files.id and fl.active = 1 and fl.recordid = items.id and fl.linktypeid = 4 group by fl.recordid ) as img,
	(select json_object_agg(parm.paramid, json_build_object('value', parm.value, 'unit', parm.unit)) from item_params parm where parm.recid = items.id group by parm.recid ) as params,
	(select jsonb_agg(to_jsonb(tags) - 'id' - 'createddate') from tags join taglinks tl on tags.id = tl.tagid where tl.recid = items.id group by tl.recid) as tags,
	(select json_object_agg(fl.linktype, json_build_object('dir', files.dir, 'ext', files.ext,'uid',files.uid,'filename',files.filename,'filetype',files.filetype,'linktype',fl.linktype)) from files join file_links fl on fl.fileid = files.id and fl.active =1 and  fl.recordid = cat.id and fl.linktypeid in(11,12) group by fl.recordid  ) as catimg,
	row_number() over (order by co.sort,isl.sort, items.sort, items.id asc) as row_number,
	co.sort as catsort,
	cat.scrolltype,
	items.parentid,
	CASE 
	WHEN (select day_dish.price from day_dish where day_dish.item_id=itemid AND work_date = current_date)>0 
		THEN (select day_dish.price from day_dish where day_dish.item_id=itemid AND work_date = current_date)   
		ELSE (select prc.price::int from pricedisc prc where prc.itemid = items.id and prc.bdate < now() and prc.edate > now() and prc.deptid in(:deptid,0) order by -prc.deptid asc limit 1 ) 
	END as price
from items
	join categories cat on items.category_id = cat.id and cat.brand = :brand and cat.visible = 1
	join categoriesorder co on co.category_id = cat.id and co.deptid =greatest (:deptid,1195) and (case when :debug = 0 then now() else (date_trunc('day', now() - interval '1 day') + interval '16 hours') end between co.bdate and co.edate)
	join inventsum isl on (isl.itemid = items.id 
		and case when :debug = 0 then now() else (date_trunc('day', now() - interval '1 day') + interval '16 hours') end between isl.bdate and isl.edate
		and isl.remainqty > 0 and isl.deptid = greatest (:deptid,1195)) --or (items.id in (50658, 50429, 50657 ))
	left join descriptions descr on items.id = descr.recid
	where items.bdate < now() and items.edate > now() and (1 = :invis or items.visible = 1)
UNION ALL
	select items.id, items.id as itemid, items.name as name,
	'1'::integer as profit,'Выгода дня' as catname, 10000 as category_id,
	coalesce (descr.text, items.name) as description,
	concat('МЕНЮ / ', cat.name, ' / ', items.name) as breadcrumbs,
	(select jsonb_agg(to_jsonb( files) - 'id' - 'path') from files
	join file_links fl on fl.fileid = files.id and fl.active = 1 and fl.recordid = items.id and fl.linktypeid = 4 group by fl.recordid ) as img,
	(select json_object_agg(parm.paramid, json_build_object('value', parm.value, 'unit', parm.unit)) from item_params parm where parm.recid = items.id group by parm.recid ) as params,
	(select jsonb_agg(to_jsonb(tags) - 'id' - 'createddate') from tags join taglinks tl on tags.id = tl.tagid where tl.recid = items.id group by tl.recid) as tags,
	(select json_object_agg(fl.linktype, json_build_object('dir', files.dir, 'ext', files.ext,'uid',files.uid,'filename',files.filename,'filetype',files.filetype,'linktype',fl.linktype)) from files join file_links fl on fl.fileid = files.id and fl.active =1 and  fl.recordid = cat.id and fl.linktypeid in(11,12) group by fl.recordid  ) as catimg,
	row_number() over (order by co.sort,isl.sort, items.sort, items.id asc) as row_number,
	-1 as catsort,
	cat.scrolltype,
	items.parentid,
	CASE 
		WHEN (select day_dish.price from day_dish where day_dish.item_id=itemid AND work_date = current_date)>0 
			THEN (select day_dish.price from day_dish where day_dish.item_id=itemid AND work_date = current_date)   
			ELSE (select prc.price::int from pricedisc prc where prc.itemid = items.id and prc.bdate < now() and prc.edate > now() and prc.deptid in(:deptid,0) order by -prc.deptid asc limit 1 ) 
		END as price
from items
	join day_dish on (items.id=day_dish.item_id )
	join categories cat on items.category_id = cat.id and cat.brand = :brand and cat.visible = 1
	join categoriesorder co on co.category_id = cat.id and co.deptid =greatest (day_dish.dept_id,1195) and (case when :debug = 0 then now() else (date_trunc('day', now() - interval '1 day') + interval '16 hours') end between co.bdate and co.edate)
	join inventsum isl on (isl.itemid = items.id 
		and case when :debug = 0 then now() else (date_trunc('day', now() - interval '1 day') + interval '16 hours') end between isl.bdate and isl.edate
		and isl.remainqty > 0 and isl.deptid = greatest (day_dish.dept_id,1195)) --or (items.id in (50658, 50429, 50657 ))
	left join descriptions descr on items.id = descr.recid
	where items.bdate < now() and items.edate > now() and (1 = :invis or items.visible = 1) AND isl.deptid=:deptid AND work_date = current_date
	order by catsort  
```

```php
$stm->execute(['deptid'=>($params->deptid ?? 0),'invis'=>$inv = (int)($_ENV['invis'] ?? 0),'brand'=>17,'debug'=>$debug]);
```
