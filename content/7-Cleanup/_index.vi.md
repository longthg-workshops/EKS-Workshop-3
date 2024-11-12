---
title: "Dọn dẹp tài nguyên"
weight: 7
chapter: false
pre: "<b> 7. </b>"
---

Trước khi xoá CloudFormation Stack của workshop, hãy đảm bảo rằng bạn đã xoá các tài nguyên lab và cụm EKS như được hướng dẫn ở phần [2.2](../../2-Prerequiste/2.2-cluster-creation/).

Sau khi thực hiện xong bước trên, chạy lệnh sau trên giao diện dòng lệnh CloudShell để xoá tài nguyên IDE:

```bash
aws cloudformation delete-stack --stack-name eks-workshop-ide
```