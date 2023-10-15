---
title: "Security Check-Up in AWS"
excerpt: "AWS 보안성 검토 스크립트"
categories:
    - Cloud Security
tags:
    - aws
toc: true
---

## 점검 기준
 - 2023 클라우드 보안 점검 가이드

## Shell Script
```bash
#!/bin/bash

CURRENT_PATH=$(pwd)
RESULT_FILE="result.txt"
now=$(date +'%Y%m%d')
VPC="vpc-09dafade257d2b05b"


# 임시 데이터 수집 파일 존재 유무
if [ -f $CURRENT_PATH/tmp* ] ; then
    echo "tmpfile exists and removed"
    rm -f $CURRENT_PATH/tmp*
else
    echo "No tmpfile."
fi

# 기존 결과 파일 백업 및 삭제
if [ -f ${RESULT_FILE} ] ; then
    echo "${RESULT_FILE} exists and back up the file."
    cp -a ${RESULT_FILE} ${RESULT_FILE}.${now}
		echo "${RESULT_FILE} is removed."
		rm -f ${RESULT_FILE}
else
		echo "${RESULT_FILE} does not exist"
fi

# 사전 자료 수집
aws iam list-users | grep -P -o '(?<="UserName": ")([^"]*)' > tmpfile.txt
aws iam list-groups | grep -P -o '(?<="GroupName": ")([^"]*)' > tmp_group.txt

# 점검 시작
echo "Wait to check a security status of your console in AWS..."
echo "==> Launching Cloud Security Checkup by DFP <==" >> $RESULT_FILE
echo ""  >> $RESULT_FILE #\r\n

echo "########## [1] 계정 관리 ##########" > $RESULT_FILE
echo "===== [1.1 - 1.3] 관리자/IAM 사용자 계정 관리 =====" >> $RESULT_FILE
for IAM in `cat $CURRENT_PATH/tmpfile.txt`
do
    echo "> User Name : $IAM" >> $RESULT_FILE
		IN_GROUP=$(aws iam list-groups-for-user --user-name $IAM | grep -P -o '(?<="GroupName": ")([^"]*)')
        TAG=$(aws iam list-user-tags --user-name $IAM --output text | awk '{ print $2, $3 }')
		echo ">> in-Group: $IN_GROUP" >> $RESULT_FILE
		echo ">> Tag: $TAG" >> $RESULT_FILE
done

echo "" >> $RESULT_FILE # \r\n
echo "===== [1.4] IAM 그룹 사용자 계정 관리 =====" >> $RESULT_FILE
for GROUP_POLICY in `cat $CURRENT_PATH/tmp_group.txt`
do
    echo "> Group Name : $GROUP_POLICY" >> $RESULT_FILE
    GRP_POLICY=$(aws iam list-attached-group-policies --group-name $GROUP_POLICY --output text | awk '{print $3}')
    echo "## Attached Policies in $GROUP_POLICY ##" >> $RESULT_FILE
    echo "$GRP_POLICY" >> $RESULT_FILE

		## AdministratorAccess 정책이 적용되어 있는 그룹에 속한 사용자만 출력 시 아래 if 주석 제거
		if [[ ${GRP_POLICY} == *"AdministratorAccess"* ]] ; then
				echo "$GROUP_POLICY" > $CURRENT_PATH/tmp_iam_admin.txt
		else
			  for IAM in `cat $CURRENT_PATH/tmpfile.txt`
		        do
		            ATTUSR=$(aws iam list-groups-for-user --user-name $IAM | grep -P -o '(?<="GroupName": ")([^"]*)')
		            if [[ ${ATTUSR} == ${GROUP_POLICY} ]] ; then
		                echo "Attached User in $GROUP_POLICY : $IAM" >> $RESULT_FILE
		            fi
		        done
		fi
done

echo "" >> $RESULT_FILE # \r\n
echo "===== [1.6] Key Pair 보관 관리 =====" >> $RESULT_FILE
KEY_PAIR=$(aws ec2 describe-key-pairs --output text | awk '{ print $3 }')
for KEY in $KEY_PAIR
do
    echo "### Key-Pair: $KEY ###" >> $RESULT_FILE
    for S3_KEY in `aws s3 ls | awk '{ print $3 }'`
    do
        echo "$ Bucket Name: $S3_KEY" >> $RESULT_FILE
        SEARCH_PEM=$(aws s3api list-objects --bucket $S3_KEY | grep -P -o '(?<="Key": ")([^"]*)' | grep ".pem")
        if [[ $SEARCH_PEM == *"$KEY"* ]] ; then
            echo "$SEARCH_PEM exists." >> $RESULT_FILE
            PEM_PERM=$(aws s3api get-object-acl --bucket $S3_KEY --key $KEY.pem --output text  | grep 'Group' | awk '{ print $2 }')
            if [ ${PEM_PERM} ] ; then
                echo "Type: ${PEM_PERM}" >> $RESULT_FILE
                echo "A Permission of Public Access exists." >> $RESULT_FILE
            else
                echo "No Public" >> $RESULT_FILE
            fi
        #데이터량이 많은 버킷 생략
        #elif [[ ${S3_KEY} == *xx* ]] ; then
		#	echo "SKIP"
        else
            echo "Key-pair does not exist in $S3_KEY." >> $RESULT_FILE
        fi
    done
done

echo "" >> $RESULT_FILE #\r\n
echo "===== [1.7] Admin Console 관리자 정책 관리 =====" >> $RESULT_FILE
IAM_ADMIN=$(cat $CURRENT_PATH/tmp_iam_admin.txt)
for IAM in `cat $CURRENT_PATH/tmpfile.txt`
do
		ATTUSR=$(aws iam list-groups-for-user --user-name $IAM | grep -P -o '(?<="GroupName": ")([^"]*)')
		if [[ ${ATTUSR} == ${IAM_ADMIN} ]] ; then
				echo "Attached User in $IAM_ADMIN : $IAM" >> $RESULT_FILE
		fi
done

echo "" >> $RESULT_FILE #\r\n
echo "===== [1.8] Admin Console 계정 Access Key 활성화 및 사용 주기 관리 =====" >> $RESULT_FILE

for IAM in `cat $CURRENT_PATH/tmpfile.txt`
do
    ACCKEY=$(aws iam list-access-keys --user-name $IAM | grep -P -o '(?<="AccessKeyId": ")([^"]*)')
    if [ ${ACCKEY} ] ; then
        ACCKEY_STATUS=$(aws iam list-access-keys --user-name $IAM | grep -P -o '(?<="Status": ")([^"]*)')
        ACCKEY_CREATE=$(aws iam list-access-keys --user-name $IAM | grep -P -o '(?<="CreateDate": ")([^"]*)')
        ACCKEY_CREATE=$(date +%Y%m%d -d $ACCKEY_CREATE)
        DATE1=$(date -d "$now" +%s)
        DATE2=$(date -d "$ACCKEY_CREATE" +%s)
        ACCKEY_LIFECYCLE=$(echo "($DATE1-$DATE2) / 86400" | bc)
        echo "$IAM's a status of access key is: $ACCKEY_STATUS" >> $RESULT_FILE #\r\n
        echo "$IAM's a created date of key: ${ACCKEY_LIFECYCLE} days ago" >> $RESULT_FILE #\r\n
    fi
done

echo "" >> $RESULT_FILE #\r\n
echo "===== [1.9] MFA (Multi-Factor Authentication) 설정 =====" >> $RESULT_FILE

MFA=$(aws iam list-virtual-mfa-devices | grep -P -o '(?<="UserName": ")([^"]*)')
echo "## Mulfi-Factor Authentication User list (Virtual) ##" >> $RESULT_FILE #\r\n
for MFA_LIST in $MFA
do
    echo " > User Name: $MFA_LIST"  >> $RESULT_FILE #\r\n
done

echo "" >> $RESULT_FILE #\r\n
echo "===== [1.10] AWS 패스워드 정책 관리 =====" >> $RESULT_FILE

aws iam get-account-password-policy --output table >> $RESULT_FILE

echo "" >> $RESULT_FILE #\r\n
echo "<[1] 계정 관리> 점검 완료." >> $RESULT_FILE

echo "" >> $RESULT_FILE #\r\n
echo "===== [3.1] 보안그룹 인/아웃바운드 ANY 설정 관리 =====" >> $RESULT_FILE

aws ec2 describe-security-groups --filter Name=ip-permission.protocol,Values='-1' Name=vpc-id,Values=$VPC --query "SecurityGroups[*].{Name:GroupName,ID:GroupId}" --output table
aws ec2 describe-security-groups --filter Name=egress.ip-permission.protocol,Values='-1' Name=vpc-id,Values=$VPC --query "SecurityGroups[*].{Name:GroupName,ID:GroupId}" --output table

echo "" >> $RESULT_FILE #\r\n
echo "===== [3.2] 보안그룹 인/아웃바운드 불필요 정책 관리 =====" >> $RESULT_FILE

aws ec2 describe-security-groups --filter Name=ip-permission.cidr,Values='0.0.0.0/0' Name=vpc-id,Values=$VPC --query "SecurityGroups[*].{Name:GroupName,ID:GroupId}" --output table
aws ec2 describe-security-groups --filter Name=egress.ip-permission.cidr,Values='0.0.0.0/0' Name=vpc-id,Values=$VPC --query "SecurityGroups[*].{Name:GroupName,ID:GroupId}" --output table

echo "" >> $RESULT_FILE #\r\n
echo "===== [3.3] 네트워크 ACL 인/아웃바운드 트래픽 정책 =====" >> $RESULT_FILE

echo "" >> $RESULT_FILE #\r\n
echo "===== [3.4] 라우팅 테이블 정책 관리 =====" >> $RESULT_FILE

RT=$(aws ec2 describe-route-tables | grep -P -o '(?<="RouteTableId": ")([^"]*)')
for ROUTETABLE in $RT
do
    echo "=== Route Table ID: $ROUTETABLE ===" >> $RESULT_FILE
    aws ec2 describe-route-tables --filter Name=route-table-id,Values=$ROUTETABLE --query "RouteTables[].Routes[*]" --output table >> $RESULT_FILE

done

인터넷 게이트웨이 연결 관리
NAT 게이트웨이 연결 관리

echo "" >> $RESULT_FILE #\r\n
echo "===== [3.7] 버킷/객체 접근 관리 =====" >> $RESULT_FILES
for S3_BUCKET in `aws s3 ls | awk '{ print $3 }'`
do
    echo "$ Bucket Name: $S3_BUCKET" >> $RESULT_FILE
    S3_ALL=$(aws s3api get-bucket-acl --bucket $S3_BUCKET --query "Grants[?(Grantee.Type == 'Group')]|[? contains(Grantee.URI, 'AllUsers')].Grantee" --output text)
    S3_AUTHUSER=$(aws s3api get-bucket-acl --bucket $S3_BUCKET --query "Grants[?(Grantee.Type == 'Group')]|[? contains(Grantee.URI, 'AuthenticatedUsers')].Grantee" --output text)
    if [ "${S3_ALL}" ] || [ "${S3_AUTHUSER}" ] ; then
        echo "This '$S3_BUCKET' can be accessed by Everyone." >> $RESULT_FILE
    else
        echo "It cannot access except OWNER." >> $RESULT_FILE
    fi
done

echo "==> Cloud Security Checkup is finished. <==" >> $RESULT_FILE
echo "Done."
```