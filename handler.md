# File handler.py

- Xử lý các tác vụ backend, bao gồm:
    - Chạy các mô hình: Dịch, tìm kiếm, ...
    - Làm việc với database: Thêm, sửa, xóa, đọc, xem
    - Chạy các thuật toán: kiểm tra trùng lặp, kiểm tra gap
    - Chạy các module: sửa rule, sửa biến, xóa rule, xóa văn bản
    - Tất cả được gói trong một class: **handler**

- Chi  tiết về các thuật toán kiểm tra trùng lặp và kiểm tra gap được trình bày ở phần tiếp theo

## Thuật toán kiểm tra trùng lặp

- Ý tưởng chính: 
    - Xét tất cả các cặp rule trong những rule chọn để check.
    - Sinh ra tất cả các trường hợp khác nhau thỏa mãn điều kiện của từng rule, dựa theo các ngưỡng đề cập trong mỗi rule.
    - Kiểm tra các trường hợp sinh ra này có bị trùng nhau hay không.
- Các hàm sử dụng:
    - **Generate_value_test_case**: Sinh giá trị cho 1 citerion của 1 quy tắc, kết quả sử dụng cho hàm **Generate_all_case_with_feature**
    - **Generate_all_case_with_feature**: Sinh tất cả các khoảng giá trị có thể có cho các biến trong bộ quy tắc cần kiểm tra
    - **Check_conditions_for_conflict**: Kiểm tra các giá trị sinh ra từ hàm **Generate_all_case_with_feature** có bị trùng lặp nhau (về mặt khoảng giá trị) hay không
    - **Generate_combinations**: Sinh ra các giá trị từ tổ hợp các biến, là hàm bổ trợ cho các hàm khác
    - **Find_parent**: Tìm biến mà biến này phụ thuộc, là hàm bổ trợ 
    - **Check_active_var**: Kiểm tra tổ hợp giá trị có thỏa mãn điều kiện nghiệp vụ hay không. Ví dụ: Nếu khách hàng có "Tuổi" thì loại khách hàng phải là "KHCN" 
    - **Check_constraint_gen**: Kiểm tra giá trị generate có thỏa mãn điều kiện nghiệp vụ về mặt giá trị hay không. Ví dụ: Nếu "Kinh nghiệm làm việc" > 2 thì "Tuổi" > 20 
    - **Check_constraint_sample**: Kiểm tra sample sinh ra có thỏa mã các điều kiện nghiệp vụ và điều kiện giá trị hay không
<!-- - Giải thích các hàm:
    - Generate_value_test_case:
        - Đầu vào: 1 criterion. 
          Ví dụ: 
          
                AGE >= 20
                 
                INCOME < 20,000,000

                COLLATERAL_TYPE in ['BĐS', 'GTCG']

                CUSTOMER_TYPE not in ['KHCN']
        - Đầu ra: 1 list thể hiện các giá trị thỏa mãn criterion đó.
          Ví dụ: 
          
                [21,20]

                [19,000,000]

                ['BĐS', 'GTCG']

                ['KHDN']

        
    - Generate_all_case_with_feature
        - Đầu vào: 2 rule id cần kiểm tra trùng lặp. Ví dụ

                R_0001: AGE >= 20 AND INCOME < 20,000,000

                R_0002: AGE >= 18 AND INCOME > 15,000,000

        - Đầu ra: 1 dict lưu các giá trị (list) thỏa mãn các criterion của từng rule. Ví dụ:

                {
                    'AGE': [[20,21],[18,19]],
                    'INCOME': [[19,000,000],[16,000,000]]
                }

    - Check_conditions_for_conflict
        - Đầu vào: Tên biến, Giá trị kiểm tra, Mã rule. Ví dụ:

                AGE, [[20,21],[18,19]], R_0001

        - Đầu ra: Những trường hợp bị trùng lặp. Ví dụ:

                [[20,21],[18,19]]

    - Generate_combinations: Sinh các tổ hợp giá trị từ danh sách biến và giá trị tương ứng
        - Đầu vào: 1 dict thể hiện danh sách các biến và các giá trị của biến. Ví dụ:

                {
                    'AGE': [20,21,18,19],
                    'INCOME': [19000000, 16000000]
                }
        
        - Đầu ra: Các tổ hợp giá trị của các biến đầu vào. Ví dụ:

                [{'AGE': 18, 'INCOME': 16000000},
                {'AGE': 18, 'INCOME': 19000000},
                {'AGE': 19, 'INCOME': 16000000},
                {'AGE': 19, 'INCOME': 19000000},
                {'AGE': 20, 'INCOME': 16000000},
                {'AGE': 20, 'INCOME': 19000000},
                {'AGE': 21, 'INCOME': 16000000},
                {'AGE': 21, 'INCOME': 19000000}] -->


## Thuật toán kiểm tra gap

- Ý tưởng chính:
    - Coi toàn bộ phần quy tắc cần kiểm tra gap là một không gian **n** chiều với **n** là số biến phân biệt trong các quy tắc kiểm tra
    - Sinh ra các tổ hợp giá trị ngẫu nhiên với **n** biến trên và kiểm tra xem giá trị đó có thỏa mãn quy tắc nào trong số các quy tắc kiểm tra không
    - Nếu không có quy tắc nào thỏa mãn thì xác định đây là một sample gap
    - Tìm các bound gần nhất tương ứng với mỗi biến của sample gap, từ đó xác định khoảng gap tương ứng

- Các hàm sử dụng:
    - **get_config_min_max**: Lấy giá trị upperbound và lowerbound của biến (nếu là biến fill)
    - **random_by_idx**: Sinh giá trị ngẫu nhiên cho biến dựa trên các ngưỡng của biến được khai báo trong các quy tắc
    - **check_constraint_gen**
    - **check_hit_in_rule**: Kiểm tra sample sinh ra có thỏa mãn quy tắc hay không
    - **consolidate_bound**: Chỉnh sửa đầu mút của sample gap phù hợp với các quy tắc kiểm tra
    





