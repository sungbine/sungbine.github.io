---
layout: post
title:  "resultMap - Extends attribute 사용"
date:   2015-06-16 19:11:00
categories: tech post
---

개발 중 발생하지 않은 에러가 발생하여 많은 시간 낭비를 한 케이스가 있어 공유하고자 한다.

ibatis 를 사용하던중(myBatis도 동일할 것이다) resultMapper를 구현하고 select 절의 result를 맵핑해서 사용하는 케이스가 있다.

이 때 resultMap의 attribute중 extends 가 있는데 이것은 기존의 resultMap을 재사용하는 케이스를 나타낸다.

예를 들어

```
<resultMap id="aaa" resultClass ="AAA">
	 <result property="receiptSeq" column="receipt_seq"/>
     <result property="purchaseApplyMId" column="purchase_apply_m_id"/>
     <result property="draftDocNo" column="draft_doc_no"/>
     <result property="bizNpartnerId" column="biz_npartner_id"/>
     <result property="npartnerId" column="npartner_id"/>
     <result property="compId" column="comp_id"/>
     <result property="contractFlag" column="contract_flag"/>
     <result property="chargeEmpNo" column="charge_emp_no"/>
     <result property="contractNm" column="contract_nm"/>
     <result property="contractClassCd" column="contract_class_cd"/>
     <result property="startDay" column="start_day"/>
</resultMap>
```

와 같은 resultMap 이 존재하고

이것을 기본으로하는 확장 resultMap을 사용하고자 할때 사용하는 attribute이다. 마치 OOP의 상속 개념과 비슷하다고 할 수있다.

```
<resultMap id="bbb" resultClass ="BBB" extends="aaa">
	 <result property="endDay" column="end_day"/>
     <result property="personalInfoOfferFlag" column="personal_info_offer_flag"/>
     <result property="contractKindCd" column="contract_kind_cd"/>
     <result property="slamContractId" column="slam_contract_id"/>
     <result property="slamContractNm" column="slam_contract_nm"/>
     <result property="privateContractFlag" column="private_contract_flag"/>
     <result property="privateContractReason" column="private_contract_reason"/>
     <result property="autoExtensionFlag" column="auto_extension_flag"/>
 
</resultMap>
```

라고 작성하면 실제로는 aaa와 bbb를 합친 

```
<resultMap id="aaabbb" resultClass ="BBB" extends="aaa">
<result property="receiptSeq" column="receipt_seq"/>
     <result property="purchaseApplyMId" column="purchase_apply_m_id"/>
     <result property="draftDocNo" column="draft_doc_no"/>
     <result property="bizNpartnerId" column="biz_npartner_id"/>
     <result property="npartnerId" column="npartner_id"/>
     <result property="compId" column="comp_id"/>
     <result property="contractFlag" column="contract_flag"/>
     <result property="chargeEmpNo" column="charge_emp_no"/>
     <result property="contractNm" column="contract_nm"/>
     <result property="contractClassCd" column="contract_class_cd"/>
     <result property="startDay" column="start_day"/>
     <result property="endDay" column="end_day"/>
     <result property="personalInfoOfferFlag" column="personal_info_offer_flag"/>
     <result property="contractKindCd" column="contract_kind_cd"/>
     <result property="slamContractId" column="slam_contract_id"/>
     <result property="slamContractNm" column="slam_contract_nm"/>
     <result property="privateContractFlag" column="private_contract_flag"/>
     <result property="privateContractReason" column="private_contract_reason"/>
     <result property="autoExtensionFlag" column="auto_extension_flag"/> 
</resultMap>
```

와 같은 resultMap 이 완성되는 것이다.


# 발생했던 문제!

문제는 이것을 이용 할때는 select 구문에서 부모의 resultMap인 aaa의 property도 모두 select구문에 들어있어야 한다는 것이다. 그렇지 않을 경우 에러가 발생한다. 
###하지만!!  
만약 쿼리의 실행결과가 존재하지않는다면! 쿼리의 실행결과가 아무것도 나오지 않는다면 에러가 발생하지 않기 때문에 정상적으로 동작하는 것으로 오해하기 쉽다.
그러나 실행결과가 1row 라도 존재하면, 바로 에러가 발생하게 된다.

select 절의 resultMap을 활용할때 꼭 extends 여부를 확인해야 한다!!

