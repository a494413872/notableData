---
tags: [lvmm]
title: mysql同步到es配置
created: '2020-11-03T07:18:23.625Z'
modified: '2020-11-03T09:36:16.985Z'
---

# mysql同步到es配置

## 数据源配置
mysql
```
数据源名称：passport数据源
code:      passportDataSource
jdbc：     jdbc:mysql://10.200.6.106:3306/passport?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&useOldAliasMetadataBehavior=true
```
ES
```
code:      passportEs
```



## 全表扫描
参数1：当前系统时间 
参数3：0
模板名|用户名|表名|数据源
-|-|-|-
passcode同步ES-全量|pass_code|pass_code_history|passportDataSource

insert触发
```
import java.util.HashMap;
import java.util.Map;
import java.util.ArrayList;
import java.util.List;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;
import java.text.DateFormat;
import org.apache.commons.lang3.StringUtils;
import org.elasticsearch.action.index.IndexResponse;
import org.elasticsearch.client.Client;
import org.elasticsearch.common.xcontent.XContentFactory;
import org.elasticsearch.index.query.MatchAllQueryBuilder;
import org.elasticsearch.index.query.QueryBuilders;
import com.lvmama.pet.sync.service.util.SyncExecutor;
import com.lvmama.pet.sync.service.util.ESWrapper;
import net.sf.json.JSONObject;
import net.sf.json.JSONArray;

import com.lvmama.comm.spring.SpringBeanProxy;
import com.lvmama.comm.sync.pojo.SyncTrigger;
import com.lvmama.pet.sync.service.SyncDataContext;

import com.lvmama.comm.bee.service.sync.SyncBaseNewService;

Integer execute(Object param, Map<String, javax.sql.DataSource> dsMap, String sourceId) {
Map map = (Map) param;
	SyncBaseNewService syncBaseService =(SyncBaseNewService)SpringBeanProxy.getBean("syncBaseNewService");


            	DateFormat foramt = new SimpleDateFormat("yyyy-MM-dd"); 
	java.util.List order_list= new ArrayList();
	SyncTrigger st = (SyncTrigger) SyncDataContext.get(SyncDataContext.TRIGGER_DATA);

	
String   create_time = st.getFieldOne();
	String num = st.getFieldThree();
	String beforDate="";
	Long code_id = 0L;
	Boolean beforeFlag = false;

    
	Integer count = Integer.parseInt(num);
   Integer start =0;
   Integer end = 4000;
   if(count==0){
	 start = 0;
   }else{
	 start = count;
   }
     //按天跑历史数据
	SimpleDateFormat dft = new SimpleDateFormat("yyyy-MM-dd");
	Date beginDate = dft.parse(create_time);
	Calendar date2 = Calendar.getInstance();
	date2.setTime(beginDate);
	date2.set(Calendar.DATE, date2.get(Calendar.DATE) - 1);
	beforDate = dft.format(date2.getTime());
   String create_time_begin = create_time+" 00:00:00";
   String create_time_end = create_time+" 23:59:59"
   
String  sql = "select  code_id from  intf_pass_code WHERE create_time  >=  '"+create_time_begin+"' and create_time  <=  '"+create_time_end+"' limit " +(start)+","+(end);
		order_list = SyncExecutor.getDefault().executeSQLQuery(sql, sourceId, dsMap.get("passportDataSource"));

//生成的多条 init 更新状态
Map<String,Object> searchParams = new HashMap<String, Object>();

searchParams.put("tableNames", ["pass_code_history"]  as String[]);
searchParams.put("userNames", ["pass_doce"]  as String[] );
searchParams.put("dataSourceId", "passportDataSource");
searchParams.put("userOp", "IN");
searchParams.put("tableOp", "IN");
searchParams.put("rows", 2000);
List<SyncTrigger> syncTriggerList = syncBaseService.findTriggerPageListWithTime(searchParams);
List<String> successList = new ArrayList<String>();

//更新状态  
for(int i=0;i<syncTriggerList.size();i++){
successList.add(syncTriggerList.get(i).getTriggerId());
}
successList.add(st.getTriggerId());
syncBaseService.updateTriggersStatus(successList,"SUCCESS");

if(null !=order_list&order_list.size()>0){
			for(int i=0;i<order_list.size();i++){
				code_id =order_list.get(i).get("code_id");
				SyncTrigger syncTrigger = new SyncTrigger();
				syncTrigger.setUserName("pass_code");
				syncTrigger.setTableName("intf_pass_code");
				syncTrigger.setPkValue(code_id+"");
				syncTrigger.setTriggerType("INSERT");
				syncTrigger.setStatus("INIT");
				syncTrigger.setDataSourceId("passportDataSource");
				syncBaseService.saveSyncTrigger(syncTrigger);

			}
        if(order_list.size()<4000){
			 beforeFlag = true;
			}
		} else{
			beforeFlag = true;
		}

		SyncTrigger syncTrigger = new SyncTrigger();
		syncTrigger.setUserName("pass_code");
		syncTrigger.setTableName("pass_code_history");
		syncTrigger.setPkValue(code_id+"");
		syncTrigger.setTriggerType("INSERT");
		syncTrigger.setStatus("INIT");
		syncTrigger.setDataSourceId("passportDataSource");
		if(beforeFlag){
			syncTrigger.setFieldOne(beforDate);
		  	syncTrigger.setFieldThree(0+"");

		}else{
			syncTrigger.setFieldOne(create_time);
		  	syncTrigger.setFieldThree((count+4000)+"");

		}
		syncTrigger.setFieldTwo("查询结果:"+order_list.size()+""+"查询范围:"+(start)+"-"+(end));

syncBaseService.saveSyncTrigger(syncTrigger);
	  return order_list.size();
}
```
job参数
```
{"userNames":["pass_code"], "tableNames":["pass_code_history"], "userOp":"IN", "tableOp":"IN","dataSourceId":"passportDataSource","triggerType":"INSERT"}
```

## detail增量
模板名|用户名|表名|数据源
-|-|-|-
passcode同步ES-增量-detail|pass_code|intf_pass_code_detail_increment|passportDataSource

insert触发
```
import java.util.HashMap;
import java.util.Map;
import java.util.ArrayList;
import java.util.List;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;
import java.text.DateFormat;
import org.apache.commons.lang3.StringUtils;

import com.lvmama.pet.sync.service.util.SyncExecutor;
import net.sf.json.JSONObject;
import net.sf.json.JSONArray;

import com.lvmama.comm.spring.SpringBeanProxy;
import com.lvmama.comm.sync.pojo.SyncTrigger;
import com.lvmama.pet.sync.service.SyncDataContext;
import com.lvmama.comm.bee.service.sync.SyncBaseNewService;

 Integer execute(Object param, Map<String, javax.sql.DataSource> dsMap, String sourceId){
        Map map = (Map) param;
        DateFormat format = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss");
        Long kpValue = 0l;
        List dataList = new ArrayList<Map<String,Object>>();
        Integer incrementNum = 2000;
        Integer resultNum = 0;
        String manualInputTime="";
        String update_time ="";
        String dataSourceId = "passportDataSource";
        String TABLE_SUFFIX ="_increment";
        String tableName = "intf_pass_code";
        String kpName ="pass_code_id";
      

        SyncBaseNewService syncBaseNewService = (SyncBaseNewService) SpringBeanProxy.getBean("syncBaseNewService");
        
        SyncTrigger st = (SyncTrigger) SyncDataContext.get(SyncDataContext.TRIGGER_DATA);

        update_time = st.getFieldOne();
           if(null==update_time||"".equals(update_time.trim())){
                
                       Calendar beforeTime = Calendar.getInstance();
                     beforeTime.add(Calendar.MINUTE, -2);
                 update_time=format.format(beforeTime.getTime());
        }
        StringBuffer sql =new StringBuffer();
        sql.append(" select pass_code_id from intf_pass_code_detail where UPDATE_DATE >= DATE_ADD(NOW(),INTERVAL -6 MINUTE) ");

         
             dataList = SyncExecutor.getDefault().executeSQLQuery(sql.toString(), sourceId, dsMap.get(dataSourceId));
         
         
        if (null != dataList && dataList.size() > 0) {
                  resultNum =dataList.size();

            for (int i = 0; i <resultNum; i++) {
                kpValue = dataList.get(i).get(kpName);
                SyncTrigger syncTrigger = new SyncTrigger();
                syncTrigger.setUserName("pass_code");
                syncTrigger.setTableName(tableName);
                syncTrigger.setPkValue(kpValue + "");
                syncTrigger.setTriggerType("INSERT");
                syncTrigger.setStatus("INIT");
                syncTrigger.setDataSourceId(dataSourceId);
                syncBaseNewService.saveSyncTrigger(syncTrigger);
            }
        }
      /*SyncTrigger syncTrigger = new SyncTrigger();
      syncTrigger.setUserName("pass_code");
      syncTrigger.setTableName(tableName+TABLE_SUFFIX);
      syncTrigger.setPkValue(kpValue + "");
      syncTrigger.setTriggerType("INSERT");
      syncTrigger.setStatus("INIT");
      syncTrigger.setDataSourceId(dataSourceId);
      syncTrigger.setFieldOne();
      syncTrigger.setFieldTwo(dataList==null?"0":dataList.size()+"");
      syncTrigger.setFieldThree(resultNum+"");
      syncBaseNewService.saveSyncTrigger(syncTrigger);*/
         
      return 1;
  }
```
job参数
```
{"userNames":["pass_code"], "tableNames":["intf_pass_code_detail_increment"], "userOp":"IN", "tableOp":"IN","dataSourceId":"passportDataSource","autoCreate":"true","triggerType":"INSERT"}
```
## port 增量
模板名|用户名|表名|数据源
-|-|-|-
passcode同步ES-增量-port|pass_code|intf_pass_port_code_increment|passportDataSource

insert触发
```
import java.util.HashMap;
import java.util.Map;
import java.util.ArrayList;
import java.util.List;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;
import java.text.DateFormat;
import org.apache.commons.lang3.StringUtils;

import com.lvmama.pet.sync.service.util.SyncExecutor;
import net.sf.json.JSONObject;
import net.sf.json.JSONArray;

import com.lvmama.comm.spring.SpringBeanProxy;
import com.lvmama.comm.sync.pojo.SyncTrigger;
import com.lvmama.pet.sync.service.SyncDataContext;
import com.lvmama.comm.bee.service.sync.SyncBaseNewService;

 Integer execute(Object param, Map<String, javax.sql.DataSource> dsMap, String sourceId){
        Map map = (Map) param;
        DateFormat format = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss");
        Long kpValue = 0l;
        List dataList = new ArrayList<Map<String,Object>>();
        Integer incrementNum = 2000;
        Integer resultNum = 0;
        String manualInputTime="";
        String update_time ="";
        String dataSourceId = "passportDataSource";
        String TABLE_SUFFIX ="_increment";
        String tableName = "intf_pass_code";
        String kpName ="code_id";
      

        SyncBaseNewService syncBaseNewService = (SyncBaseNewService) SpringBeanProxy.getBean("syncBaseNewService");
        
        SyncTrigger st = (SyncTrigger) SyncDataContext.get(SyncDataContext.TRIGGER_DATA);

        update_time = st.getFieldOne();
           if(null==update_time||"".equals(update_time.trim())){
                
                       Calendar beforeTime = Calendar.getInstance();
                     beforeTime.add(Calendar.MINUTE, -2);
                 update_time=format.format(beforeTime.getTime());
        }
        StringBuffer sql =new StringBuffer();
        sql.append("select code_id from intf_pass_port_code where update_date >= DATE_ADD(NOW(),INTERVAL -10 MINUTE)");

         
             dataList = SyncExecutor.getDefault().executeSQLQuery(sql.toString(), sourceId, dsMap.get(dataSourceId));
         
         
        if (null != dataList && dataList.size() > 0) {
                  resultNum =dataList.size();

            for (int i = 0; i <resultNum; i++) {
                kpValue = dataList.get(i).get(kpName);
                SyncTrigger syncTrigger = new SyncTrigger();
                syncTrigger.setUserName("pass_code");
                syncTrigger.setTableName(tableName);
                syncTrigger.setPkValue(kpValue + "");
                syncTrigger.setTriggerType("INSERT");
                syncTrigger.setStatus("INIT");
                syncTrigger.setDataSourceId(dataSourceId);
                syncBaseNewService.saveSyncTrigger(syncTrigger);
            }
        }
      /*SyncTrigger syncTrigger = new SyncTrigger();
      syncTrigger.setUserName("pass_code");
      syncTrigger.setTableName(tableName+TABLE_SUFFIX);
      syncTrigger.setPkValue(kpValue + "");
      syncTrigger.setTriggerType("INSERT");
      syncTrigger.setStatus("INIT");
      syncTrigger.setDataSourceId(dataSourceId);
      syncTrigger.setFieldOne();
      syncTrigger.setFieldTwo(dataList==null?"0":dataList.size()+"");
      syncTrigger.setFieldThree(resultNum+"");
      syncBaseNewService.saveSyncTrigger(syncTrigger);*/
         
      return 1;
  }
```
job参数
```
{"userNames":["pass_code"], "tableNames":["intf_pass_port_code_increment"], "userOp":"IN", "tableOp":"IN","dataSourceId":"passportDataSource","autoCreate":"true","triggerType":"INSERT"}
```

## 增量
模板名|用户名|表名|数据源
-|-|-|-
passcode同步ES-增量|pass_code|intf_pass_code_increment|passportDataSource

insert触发
```
import java.util.HashMap;
import java.util.Map;
import java.util.ArrayList;
import java.util.List;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;
import java.text.DateFormat;
import org.apache.commons.lang3.StringUtils;

import com.lvmama.pet.sync.service.util.SyncExecutor;
import net.sf.json.JSONObject;
import net.sf.json.JSONArray;

import com.lvmama.comm.spring.SpringBeanProxy;
import com.lvmama.comm.sync.pojo.SyncTrigger;
import com.lvmama.pet.sync.service.SyncDataContext;
import com.lvmama.comm.bee.service.sync.SyncBaseNewService;

 Integer execute(Object param, Map<String, javax.sql.DataSource> dsMap, String sourceId){
        Map map = (Map) param;
        DateFormat format = new SimpleDateFormat("yyyy/MM/dd HH:mm:ss");
        Long kpValue = 0l;
        List dataList = new ArrayList<Map<String,Object>>();
        Integer incrementNum = 2000;
        Integer resultNum = 0;
        String manualInputTime="";
        String update_time ="";
        String dataSourceId = "passportDataSource";
        String TABLE_SUFFIX ="_increment";
        String tableName = "intf_pass_code";
        String kpName ="code_id";
      

        SyncBaseNewService syncBaseNewService = (SyncBaseNewService) SpringBeanProxy.getBean("syncBaseNewService");
        
        SyncTrigger st = (SyncTrigger) SyncDataContext.get(SyncDataContext.TRIGGER_DATA);

        update_time = st.getFieldOne();
           if(null==update_time||"".equals(update_time.trim())){
                
                       Calendar beforeTime = Calendar.getInstance();
                     beforeTime.add(Calendar.MINUTE, -2);
                 update_time=format.format(beforeTime.getTime());
        }
        StringBuffer sql =new StringBuffer();
        sql.append(" select code_id from intf_pass_code where update_time >= DATE_ADD(NOW(),INTERVAL -10 MINUTE)");

             dataList = SyncExecutor.getDefault().executeSQLQuery(sql.toString(), sourceId, dsMap.get(dataSourceId));
         
         
        if (null != dataList && dataList.size() > 0) {
                  resultNum =dataList.size();

            for (int i = 0; i <resultNum; i++) {
                kpValue = dataList.get(i).get(kpName);
                SyncTrigger syncTrigger = new SyncTrigger();
                syncTrigger.setUserName("pass_code");
                syncTrigger.setTableName(tableName);
                syncTrigger.setPkValue(kpValue + "");
                syncTrigger.setTriggerType("INSERT");
                syncTrigger.setStatus("INIT");
                syncTrigger.setDataSourceId(dataSourceId);
                syncBaseNewService.saveSyncTrigger(syncTrigger);
            }
        }
      /*SyncTrigger syncTrigger = new SyncTrigger();
      syncTrigger.setUserName("pass_code");
      syncTrigger.setTableName(tableName+TABLE_SUFFIX);
      syncTrigger.setPkValue(kpValue + "");
      syncTrigger.setTriggerType("INSERT");
      syncTrigger.setStatus("INIT");
      syncTrigger.setDataSourceId(dataSourceId);
      syncTrigger.setFieldOne();
      syncTrigger.setFieldTwo(dataList==null?"0":dataList.size()+"");
      syncTrigger.setFieldThree(resultNum+"");
      syncBaseNewService.saveSyncTrigger(syncTrigger);*/
         
      return 1;
  }
```
job参数
```
{"userNames":["pass_code"], "tableNames":["intf_pass_code"], "userOp":"IN", "tableOp":"IN","dataSourceId":"passportDataSource","triggerType":"INSERT"}
```

## 单条同步脚本
模板名|用户名|表名|数据源
-|-|-|-
passcode同步ES|pass_code|intf_pass_code|passportDataSource
源脚本
```
SELECT
	t.code_id AS "code_code_id",
	t.serialno AS "code_serialno",
	DATE_FORMAT( t.create_time, '%Y-%m-%d %H:%i:%s' ) AS "code_create_time",
	t.STATUS AS "code_status",
	t.CODE AS "code_code",
	t.add_code AS "code_add_code",
	t.add_code_md5 AS "code_add_code_md5",
	t.mobile AS "code_mobile",
	t.sms_content AS "code_sms_content",
	t.ext_id AS "code_ext_id",
	t.url AS "code_url",
	t.object_id AS "code_object_id",
	t.object_type AS "code_object_type",
	t.status_no AS "code_status_no",
	t.status_explanation AS "code_status_explanation",
	t.terminal_content AS "code_terminal_content",
	t.code_image AS "code_code_image",
	t.send_sms AS "code_send_sms",
	DATE_FORMAT( t.update_time, '%Y-%m-%d %H:%i:%s' ) AS "code_update_time",
	t.send_orderid AS "code_send_orderid",
	t.order_id AS "code_order_id",
	DATE_FORMAT( t.reapply_time, '%Y-%m-%d %H:%i:%s' ) AS "code_reapply_time",
	t.reapply_count AS "code_reapply_count",
	t.code_total AS "code_code_total",
	DATE_FORMAT( t.failed_time, '%Y-%m-%d %H:%i:%s' ) AS "code_failed_time",
	t.disabled AS "code_disabled",
	DATE_FORMAT( t.create_datetime, '%Y-%m-%d %H:%i:%s' ) AS "code_create_datetime",
	t.provider_id AS "code_provider_id",
	t.provider_name AS "code_provider_name",
	t.distributor_id AS "code_distributor_id",
	t.file_id AS "code_file_id",
	t.content AS "code_content",
	t.lvurl_index AS "code_lvurl_index",
	t.refund_item_info AS "code_refund_item_info",
	t.code_image_flag AS "code_code_image_flag",
	t.hoard_flag AS "code_hoard_flag",
	t.hoard_code_batch AS "code_hoard_code_batch",
	t.export_code_batch AS "code_export_code_batch",
	t.hoard_code_status AS "code_hoard_code_status",
	t.hoard_mobile AS "code_hoard_mobile",
	t.pass_code_return_type AS "code_pass_code_return_type",
	t.code_max_process_times AS "code_code_max_process_times",
	t.code_processed_times AS "code_code_processed_times",
	t.code_detail_status AS "code_code_detail_status",
	t.batch_no AS "code_batch_no",
	t.has_new_list AS "code_has_new_list",
	(select is_auto_perform from intf_pass_provider p where p.provider_id=t.provider_id) as "provider_is_auto_perform",
	d.pass_code_detail_id AS "detail_pass_code_detail_id",
	d.pass_code_id AS "detail_pass_code_id",
	DATE_FORMAT( d.visite_time, '%Y-%m-%d %H:%i:%s' ) AS "detail_visite_time",
	d.user_id AS "detail_user_id",
	d.suppgoods_id AS "detail_suppgoods_id",
	d.provider_goods_code AS "detail_provider_goods_code",
	d.prod_manager AS "detail_prod_manager",
	d.supplier_id AS "detail_supplier_id",
	d.supplier_name AS "detail_supplier_name",
	d.distributor_code AS "detail_distributor_code",
	d.distributor_name AS "detail_distributor_name",
	d.content AS "detail_content",
	d.redestroy_count AS "detail_redestroy_count",
	d.distribution_channel AS "detail_distribution_channel",
	d.hoard_voucher AS "detail_hoard_voucher",
	d.hoard_hashid AS "detail_hoard_hashid",
	d.hoard_full_name AS "detail_hoard_full_name",
	d.hoard_act_id AS "detail_hoard_act_id",
	DATE_FORMAT( d.create_date, '%Y-%m-%d %H:%i:%s' ) AS "detail_create_date",
	DATE_FORMAT( d.update_date, '%Y-%m-%d %H:%i:%s' ) AS "detail_update_date" 
FROM
	intf_pass_code t
	LEFT JOIN intf_pass_code_detail d ON t.code_id = d.pass_code_id 
WHERE
	t.code_id = #_ID_#
```
insert触发
```
import java.util.HashMap;
import java.util.Map;
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;

import org.apache.commons.lang3.StringUtils;
import org.elasticsearch.action.index.IndexResponse;
import org.elasticsearch.client.Client;
import org.elasticsearch.common.xcontent.XContentFactory;
import org.elasticsearch.index.query.MatchAllQueryBuilder;
import org.elasticsearch.index.query.QueryBuilders;
import com.lvmama.pet.sync.service.util.SyncExecutor;
import com.lvmama.pet.sync.service.util.ESWrapper;


Integer execute(Object param, Map<String, javax.sql.DataSource> dsMap, String sourceId) {
            Map map = (Map) param;
            Client client = ESWrapper.getClient("passportEs");
            String codeId=map.get("code_code_id").toString();
            String sql = "SELECT p.pass_point_id AS pass_point_id, p.code_id AS code_id, p.STATUS AS status, DATE_FORMAT( p.used_time, '%Y-%m-%d %H:%i:%s' ) AS used_time, p.terminal_content AS terminal_content, DATE_FORMAT( p.valid_time, '%Y-%m-%d %H:%i:%s' ) AS valid_time, DATE_FORMAT( p.invalid_time, '%Y-%m-%d %H:%i:%s' ) AS invalid_time, p.ext_id AS ext_id, p.status_no AS status_no, p.status_explanation AS status_explanation, p.invalid_date AS invalid_date, p.invalid_date_memo AS invalid_date_memo, p.disabled AS disabled, DATE_FORMAT( p.create_date, '%Y-%m-%d %H:%i:%s' ) AS create_date, DATE_FORMAT( p.update_date, '%Y-%m-%d %H:%i:%s' ) AS update_date, p.code_content AS code_content FROM intf_pass_port_code p WHERE p.code_id = "+codeId;
            java.util.List code_list = SyncExecutor.getDefault().executeSQLQuery(sql, sourceId, dsMap.get("passportDataSource"));
            if(null != code_list && code_list.size() > 0){
            		map.put("pass_port_code",code_list);
        		}

            IndexResponse response = client.prepareIndex("code", "data", codeId)
                    .setSource(map)
                    .execute().actionGet();

		  return 1;
	
		
}
```
job参数
```
{"userNames":["pass_code"], "tableNames":["intf_pass_code"], "userOp":"IN", "tableOp":"IN","dataSourceId":"passportDataSource","triggerType":"INSERT"}
```

