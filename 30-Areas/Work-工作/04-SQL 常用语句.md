---
title: SQL 常用语句
created: 2026-03-11
tags:
  - work
  - sql
  - reference
aliases:
  - SQL 速查
---
## 📝 工作常用查询

<!-- 记录实际工作中常用的特定查询 -->
重跑报告
```sql
-- 光谷
update VocaPatient set canPushReport = 1,StartFinalExamTime = null,ID_FinalExam = null,FinalExamName = null,FinalExamTime = null,ID_FinalJudgment = null,FinalJudgmentName = null,FinalJudgmentTime = null
where IS_State  = '5' and CheckFinishTime  >= '2026-02-23 00:00:00' and CheckFinishTime < '2026-03-12 00:00:00';

-- 河南
update VocaPatient set IS_State ='5',canPushReport=1,generatePriority =-5,StartFinalExamTime =null where canPushReport =2 and  PatientCode  in 
(
select PatientCode 
--SELECT ID_Patient ,PatientCode,PatientName ,IS_State ,canPushReport ,CreateTime ,RegisterTime,isCalendarData ,generatePriority ,StartFinalExamTime,ID_FinalExam ,FinalExamName ,FinalExamTime
from VocaPatient 
where IS_State <='5' and StartFinalExamTime is null and FinalExamName in 
('任中豪', '黄莉茹', '马丽敏', '刘慧', '郑植元', '马亚萍','李进营', '郭真真', '许琨', '王琦', '雷淼', '袁莉莉', '代佳', '李旭')
and ISNULL(GenerateFailedNumber,0) <4 and isCalendarData != 1 and PatientCode is not null 
);
-- 舟山（在体软数据库查，查到patientcode列表到主检找）
select * from VocaPatient where PreFinalExam = 1 and IS_State < 6 AND patientname not like '%测试%' AND PreFinalExamTime > '2026-01-01 00:00:00';

update VocaPatient set canPushReport = 1,StartFinalExamTime = null,ID_FinalExam = null,FinalExamName = null,FinalExamTime = null,ID_FinalJudgment = null,FinalJudgmentName = null,FinalJudgmentTime = null from VocaPatient vp where vp.PatientCode in('2601190029') and vp.IS_State < 6;

-- 测试
update VocaPatient
set canPushReport = 1,
	StartFinalExamTime = NULL,
    IS_State = '5'
-- select canpushreport, *
from vocapatient where PatientCode like '测试%';
```

```sql
-- 备份
SELECT *
INTO DictConclusion_260311_bak
FROM DictConclusion;

-- 恢复
INSERT INTO DictConclusion
SELECT *
FROM DictConclusion__bak;

-- 将要更新的数据使用csv插入为临时表
CREATE TABLE tmp_update_from_csv (
ID_Conclusion int NOT NULL,
ConclusionName varchar(MAX) COLLATE Chinese_PRC_CI_AS NULL,
BriefCode varchar(MAX) COLLATE Chinese_PRC_CI_AS NULL,
Suggest varchar(MAX) COLLATE Chinese_PRC_CI_AS NULL,
ID_SevereLevel int NULL,
InterfaceCode1 varchar(MAX) COLLATE Chinese_PRC_CI_AS NULL,
InterfaceCode2 varchar(MAX) COLLATE Chinese_PRC_CI_AS NULL,
IS_MergeLevel float NOT NULL,
ID_Operate int NULL,
OperateName varchar(MAX) COLLATE Chinese_PRC_CI_AS NULL,
OperateTime varchar(MAX) COLLATE Chinese_PRC_CI_AS NULL,
Note varchar(MAX) COLLATE Chinese_PRC_CI_AS NULL,
IS_State varchar(MAX) COLLATE Chinese_PRC_CI_AS NULL,
ID_Disease varchar(MAX) COLLATE Chinese_PRC_CI_AS NULL,
ICDCode varchar(MAX) COLLATE Chinese_PRC_CI_AS NULL,
ConclusionType int NULL,
BodyPart varchar(MAX) COLLATE Chinese_PRC_CI_AS NULL,
IS_Ocd int NULL,
IS_ShowNumer varchar(MAX) COLLATE Chinese_PRC_CI_AS NULL,
analysis nvarchar(MAX) COLLATE Chinese_PRC_CI_AS NULL,
human_system varchar(MAX) COLLATE Chinese_PRC_CI_AS NULL,
isMaster int NULL,
masterConclusionId int NULL,
isDefaultGeneralMaster int NULL,
orderNum int NULL,
healthProblemType int NULL,
conditionNumForMerge int NULL,
conditionNumForMasterMerge int NULL,
hospital varchar(MAX) COLLATE Chinese_PRC_CI_AS NULL,
assistanceInfo varchar(MAX) COLLATE Chinese_PRC_CI_AS NULL,
thirdIdConclusion nvarchar(MAX) COLLATE Chinese_PRC_CI_AS NULL,
suggestPinyin varchar(MAX) COLLATE Chinese_PRC_CI_AS NULL,
matchPriority int NULL,
defaultSuggest varchar(MAX) COLLATE Chinese_PRC_CI_AS NULL,
analysisType varchar(MAX) COLLATE Chinese_PRC_CI_AS DEFAULT '0' NULL
);


-- 先确认数据在更新
SELECT *
FROM DictConclusion dc  
JOIN tmp_update_from_csv t  
ON dc.ConclusionName = t.ConclusionName;
    
UPDATE dc  
SET  
dc.analysis = t.analysis,  
dc.Suggest = t.Suggest  
FROM DictConclusion dc  
JOIN tmp_update_from_csv t  
ON dc.ConclusionName = t.ConclusionName;

INSERT INTO DictConclusion (ConclusionName, analysis, Suggest)  
SELECT  
t.ConclusionName,  
t.analysis,  
t.Suggest  
FROM tmp_update_from_csv t  
LEFT JOIN DictConclusion dc  
ON t.ConclusionName = dc.ConclusionName  
WHERE dc.ConclusionName IS NULL;
```

```sql
UPDATE VocaPatient 
SET 
    canPushReport = 1,           -- 设为待生成状态
    StartFinalExamTime = NULL,   -- 【关键】清除开始时间，否则不会被拉取
    IS_State = '5',              -- 确保状态回到总检阶段
    generatePriority = -2,       -- (可选) 设为负数可以提高优先级，优先重跑
    isRunning = 0,               -- (可选) 防止死锁
    GenerateFailedNumber = 0     -- (可选) 重置失败计数
WHERE patientCode in ('174494_4','133870_4','200911_2','151134_5','200968_2','162231_5','228841_1');
```
