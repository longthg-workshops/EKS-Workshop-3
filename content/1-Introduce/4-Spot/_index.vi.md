---
title: "Amazon EC2 Spot instance"
weight: 4
chapter: false
pre: "<b> 1.4 </b>"
---

Một **Spot Instance** sử dụng dung lượng EC2 dư thừa có sẵn với giá thấp hơn giá **On-Demand**. Bởi vì **Spot Instances** cho phép bạn yêu cầu các máy EC2 không sử dụng với giảm giá đáng kể, bạn có thể giảm chi phí Amazon EC2 của mình đáng kể. Giá hàng giờ cho một **Spot Instance** được gọi là **Spot Price** (giá điểm). Giá điểm của mỗi loại **instance** trong mỗi AZ được đặt bởi Amazon EC2, và được điều chỉnh từ từ dựa trên cung-cầu dài hạn cho các **Spot Instances**. **Spot Instance** của bạn chạy bất cứ khi nào có dung lượng có sẵn.

**Spot Instances** là lựa chọn phù hợp cho các ứng dụng linh hoạt, phi trạng thái và có khả năng chịu lỗi. Điều này bao gồm các công việc theo lô (batch jobs), luyện máy học, các công việc **ETL** dữ liệu lớn như Apache Spark, các ứng dụng xử lý hàng đợi và các **API** endpoint phi trạng thái.

Bởi vì **Spot** là dung lượng EC2 dư, có thể thay đổi theo thời gian, chúng tôi khuyến nghị bạn sử dụng dung lượng **Spot** cho các công việc có khả năng chịu gián đoạn. Cụ thể hơn, dung lượng **Spot** phù hợp cho các công việc có thể chịu được các khoảng thời gian không có sẵn dung lượng.