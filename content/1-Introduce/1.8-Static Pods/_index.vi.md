---
title: "Static Pods"
date: "`r Sys.Date()`"
weight: 8
chapter: false
pre: "<b>  1.8 </b>"
---

Trong phần này, chúng ta sẽ xem xét về **Static Pods**.

Làm thế nào để cung cấp một tập tin định nghĩa **pod** cho **kubelet** mà không cần có **kube-apiserver**? Bạn có thể cấu hình **kubelet** để đọc các tập tin định nghĩa **pod** từ một thư mục trên máy chủ được chỉ định để lưu thông tin về các **pod**.

#### Cấu hình Static Pod
Thư mục được chỉ định có thể là bất kỳ thư mục nào trên máy chủ và vị trí của thư mục đó được truyền vào **kubelet** dưới dạng một tùy chọn trong quá trình chạy dịch vụ.

Tùy chọn được gọi là `--pod-manifest-path`.

#### Cách cấu hình static pod khác
Thay vì chỉ định tùy chọn trực tiếp trong tệp **kubelet.service**, bạn có thể cung cấp một đường dẫn đến một tệp cấu hình khác bằng cách sử dụng tùy chọn cấu hình và xác định đường dẫn thư mục là **staticPodPath** trong tệp đó.

#### Xem các static pod
Để xem các **static pod**

```
$ docker ps
```

**Kubelet** có thể tạo cả hai loại **pod** - các **static pod** và các **pod** từ máy chủ api cùng một lúc.

#### Static Pods - Use Case

Static Pods trong Kubernetes là các pod được quản lý trực tiếp bởi kubelet daemon trên một node cụ thể, không qua apiserver. Điều này có nghĩa là chúng không thể được quản lý hoặc điều khiển từ xa bởi các bộ phận khác của Kubernetes như `kubectl` hoặc apiserver. Static Pods thường được sử dụng trong các trường hợp cụ thể, ví dụ như khởi chạy các thành phần hệ thống của Kubernetes như kube-proxy, etcd, hoặc kube-apiserver chính trên cluster.

#### 1. **Use Case: Quản lý cụm Kubernetes trên các môi trường không có hoặc giới hạn truy cập mạng**

Khi triển khai K#ubernetes trong một môi trường không có hoặc có truy cập mạng giới hạn (ví dụ: mạng riêng, môi trường sản xuất ngoại tuyến), việc sử dụng Static Pods cho phép các nhà quản trị triển khai và quản lý các thành phần cốt lõi của Kubernetes mà không cần kết nối mạng đến apiserver. Điều này giúp đảm bảo rằng cụm có thể tiếp tục hoạt động ngay cả khi mất kết nối mạng.

#### 2. **Use Case: Bootstrapping cụm Kubernetes**

Trong quá trình khởi tạo cụm Kubernetes, các Static Pods thường được sử dụng để chạy các thành phần cốt lõi của cụm như kube-apiserver, etcd, và controller manager trước khi cụm có thể tự quản lý qua apiserver. Điều này giúp khởi tạo cụm một cách tự động mà không cần can thiệp thủ công.

#### 3. **Use Case: Chạy các ứng dụng cần chế độ khả dụng cao**

Vì Static Pods được quản lý trực tiếp bởi kubelet, chúng sẽ được tự động khởi động lại bởi kubelet nếu chúng thất bại. Điều này có thể hữu ích cho các ứng dụng cần độ khả dụng cao và tự phục hồi mà không phụ thuộc vào các cơ chế điều khiển của Kubernetes thông thường.

#### 4. **Use Case: Triển khai đơn giản trên môi trường có tài nguyên hạn chế**

Trong một số trường hợp, việc triển khai một cụm Kubernetes đầy đủ có thể không khả thi do hạn chế về tài nguyên hệ thống. Trong những trường hợp này, việc sử dụng Static Pods cho phép triển khai các ứng dụng một cách đơn giản mà không cần các thành phần điều khiển phức tạp, giảm bớt yêu cầu về tài nguyên.

#### Cách tạo Static Pod

Để tạo Static Pod, bạn cần đặt tệp YAML hoặc JSON mô tả pod vào một thư mục mà kubelet được cấu hình để theo dõi (thông qua tùy chọn `--pod-manifest-path` của kubelet). Kubelet sẽ tự động tạo và quản lý các Pod dựa trên các tệp mô tả này.

Trong kịch bản triển khai thực tế, việc sử dụng Static Pods yêu cầu sự hiểu biết vững chắc về cách kubelet và các thành phần khác của Kubernetes hoạt động. Mặc dù chúng không phải là giải pháp phù hợp cho mọi tình huống, nhưng Static Pods cung cấp một công cụ linh hoạt và mạnh mẽ cho các tình huống đặc biệt.

#### Static Pods vs DaemonSets

| Tiêu Chí          | Static Pods                          | DaemonSets                              |
|-------------------|--------------------------------------|-----------------------------------------|
| **Khởi tạo**      | Khởi tạo bởi kubelet trên mỗi node   | Khởi tạo bởi API server qua một bản mô tả DaemonSet |
| **Quản lý**       | Quản lý trực tiếp bởi kubelet trên node mà nó chạy | Quản lý thông qua API server với các cơ chế điều khiển khác |
| **Cập nhật**      | Cập nhật bằng cách sửa đổi file cấu hình trên node | Cập nhật thông qua cập nhật bản mô tả DaemonSet trong API server |
| **Phạm vi hoạt động** | Chỉ chạy trên một node cụ thể, không được quản lý bởi API server | Chạy trên tất cả các node hoặc các node được chọn lọc thông qua mô hình nhãn (labels) |
| **Mục đích sử dụng** | Phù hợp cho các dịch vụ cấp thấp và cơ bản của hệ thống, như kube-proxy, kube-dns | Phù hợp cho các dịch vụ cần chạy trên mỗi node, như log collectors, monitoring agents |
| **Tự động khởi động lại** | Kubelet sẽ tự động khởi động lại Pod nếu nó bị lỗi | API server đảm bảo Pod luôn được chạy trên các node như mô tả |
| **Tính linh hoạt** | Ít linh hoạt hơn, vì cần sửa đổi file cấu hình và quản lý trực tiếp trên mỗi node | Linh hoạt hơn, có thể dễ dàng cập nhật và quản lý từ một nơi central thông qua API server |
| **Khả năng quản lý** | Quản lý tập trung khó khăn hơn do cần can thiệp trực tiếp vào mỗi node | Dễ dàng quản lý tập trung thông qua kubectl và các công cụ khác qua API server |

#### Tài liệu tham khảo Kubernetes
[https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)
