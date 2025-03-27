# Spring Boot Task API

Ứng dụng RESTful API đơn giản xây dựng bằng Spring Boot để quản lý danh sách công việc (Tasks).
Dự án này được thiết kế để minh họa cách triển khai CI/CD với GitLab.

## Tính năng

- Tạo, xem, cập nhật và xóa công việc (CRUD)
- Tìm kiếm công việc theo từ khóa
- Lọc công việc theo trạng thái (hoàn thành/chưa hoàn thành)
- Đánh dấu công việc là hoàn thành

## Công nghệ sử dụng

- Spring Boot 3.4.4
- Spring Data JPA
- H2 Database (cơ sở dữ liệu trong bộ nhớ)
- Lombok
- Jakarta EE
- Maven
- GitLab CI/CD

## Yêu cầu hệ thống

- Java 17 hoặc cao hơn
- Maven 3.6+

## Cài đặt và chạy

1. Clone repository
```bash
git clone https://gitlab.com/your-username/taskapi.git
cd taskapi