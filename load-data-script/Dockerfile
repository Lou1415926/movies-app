# Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
# SPDX-License-Identifier: MIT-0
FROM public.ecr.aws/amazonlinux/amazonlinux:latest
RUN yum install python3.7 -y && curl -O https://bootstrap.pypa.io/get-pip.py && python3 get-pip.py && yum update -y
RUN yum install -y wget
RUN yum install -y unzip
COPY . /app
WORKDIR /app
RUN pip3 install boto3
CMD bash ./LoadData.sh
