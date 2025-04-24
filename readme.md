PROVISION AN AZURE SYNAPSE ANALYTICS WORKSPACE
Azure Synapse Analytics là một nền tảng phân tích dữ liệu hợp nhất, cho phép ta xử lý dữ liệu từ đầu đến cuối: nhập, khám phá, phân tích và trực quan hóa dữ liệu. Trong bài lab này, ta sẽ tìm hiểu tổng quan về các tính năng của Azure Synapse Analytics.
Thời gian thực hiện: Khoảng 60 phút
 Yêu cầu: Tài khoản Azure có quyền quản trị (admin)
Provision an Azure Synapse Analytics workspace
1.    Truy cập trang https://portal.azure.com và đăng nhập.
2.    Mở Azure Cloud Shell bằng cách nhấn vào biểu tượng >_ ở góc trên bên phải.
3.    Chọn môi trường PowerShell và tạo storage nếu được yêu cầu.
 
 
 
 
 
4.    Nhập các lệnh sau để tải mã nguồn lab:
rm -r dp-000 -f
git clone https://github.com/MicrosoftLearning/mslearn-synapse dp-000
cd dp-000/Allfiles/Labs/01
ls
vim setup.json
 
Sửa spark version 3.1 thành 3.4
 
./setup.ps1
 
•	Nếu có nhiều subscription, ta sẽ được chọn subscription.
•	Nhập mật khẩu để tạo SQL pool (hãy ghi nhớ mật khẩu này vì sẽ sử dụng sau).
 
Sau khi chạy xong script, quá trình khởi tạo mất khoảng 20 phút.
 
 
 
 
 
Các lỗi này do bcp (bulk copy program - dùng để sao chép dữ liệu hàng loạt giữa một phiên bản Microsoft SQL Server và tệp dữ liệu theo định dạng do người dùng chỉ định) không được cài đặt trên powershell azure portal. Phần sẽ khắc phục lỗi này bằng cách cài đặt powershell azure và azure CLI trên Powershell của Windows. 
Explore Synapse Studio
1.    Quay lại Azure Portal, mở Resource Group có tên dạng dp000-xxxxx. (dp000-4mu2nfc)
 
 
2.    Tìm và mở Synapse workspace, chọn Open Synapse Studio.
 
 
 
Các trang quan trọng trong Synapse Studio (mở menu bên trái bằng biểu tượng ››):
•	Data: Gồm 2 tab:
o	Workspace: Các CSDL trong workspace.
o	Linked: Các nguồn dữ liệu liên kết (ví dụ Azure Data Lake Storage).
 
 
•	Develop: Nơi tạo và lưu các script (SQL, notebooks, v.v).
 
•	Integrate: Quản lý pipeline để nhập và xử lý dữ liệu.
 
•	Monitor: Theo dõi và kiểm tra lịch sử thực thi pipeline.
 
•	Manage: Quản lý SQL pool, Spark pool, Data Explorer pool:
o	SQL pools:
	Built-in: serverless SQL pool.
	sqlxxxxxxx: dedicated SQL pool.
 
o	Apache Spark pool: sparkxxxxxxx
 
o	Data Explorer pool: adxxxxxxxx
 
Ingest data with a pipeline
1.    Trong Synapse Studio, chọn Home > Ingest.
 
2.    Ở bước Properties:
o   Chọn: Built-in copy task và Run once now → Next >
 
3.    Source step > Dataset substep:
o   Source type: All
o   Connection: Tạo mới → chọn HTTP
 
Chọn Continue>
o   Thiết lập:
§  Name: Products
§  Description: Product list via HTTP
§  Base URL: https://raw.githubusercontent.com/MicrosoftLearning/mslearn-synapse/master/Allfiles/Labs/01/adventureworks/products.csv
§  Server Certificate Validation: Enable
§  Authentication: Anonymous → Create
 
Thành công:
 
4.    Giữ nguyên các thiết lập còn lại, chọn Next >
 
5.    Bấm Preview data để xem dữ liệu, sau đó đóng cửa sổ preview.
 
6. File format settings 
o   Format: DelimitedText
o   Column delimiter: ,
o   Row delimiter: \n
o   First row as header: Selected
o   Compression: None → Next >
 
7.    Cấu hình Destination:
•	 Destination type: Azure Data Lake Storage Gen2
•	 Connection: Chọn connection đã có sẵn (được tạo sẵn khi tạo workspace)
•	 Thiết lập:
o   Folder path: files/product_data
o   File name: products.csv
o   Copy behavior: None
 
Chọn Next >
o   File format: DelimitedText
o   Column delimiter: ,
o   Add header to file: Selected
 
Chọn Next>
7. Settings:   
•	Task name: Copy products
•	Task description: Copy products data
 
→ Next >
7.    Hoàn tất wizard và nhấn Finish để chạy pipeline.
 
Chọn Next>
 
Ta có thể vào tab Monitor để xem tiến trình sao chép có thành công không.
Theo dõi pipeline
1.    Vào Monitor > Pipeline runs
2.    Đợi cho đến khi trạng thái là Succeeded
 
3.    Vào Integrate, xác minh pipeline Copy products đã được tạo
  
4. Xem dữ liệu đã nhập
1.    Vào Data > Linked tab
2.    Mở rộng Product Files cho đến khi thấy thư mục product_data chứa file products.csv
 
3.    Nhấp chuột phải vào products.csv, chọn Preview để xem dữ liệu
 
 
Use a serverless SQL pool to analyze data
Sau khi ta đã nạp dữ liệu vào không gian làm việc của mình, ta có thể sử dụng Synapse Analytics để truy vấn và phân tích dữ liệu đó. Một trong những cách phổ biến nhất để truy vấn dữ liệu là sử dụng SQL, và trong Synapse Analytics, ta có thể dùng serverless SQL pool để chạy mã SQL với dữ liệu trong data lake.
Trong Synapse Studio, nhấp chuột phải vào tệp products.csv trong bộ lưu trữ tệp của không gian làm việc Synapse, trỏ đến New SQL script và chọn Select TOP 100 rows.
 
 
Trong khung SQL Script 1 mới mở, xem mã SQL được tạo tự động, có thể giống như sau:
-- This is auto-generated code
SELECT
    TOP 100 *
FROM
    OPENROWSET(
        BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/product_data/products.csv',
        FORMAT = 'CSV',
        PARSER_VERSION='2.0'
    ) AS [result]
Mã trên mở một tập bản ghi từ tệp văn bản ta đã nhập và truy xuất 100 dòng đầu tiên của dữ liệu.
Trong danh sách Connect to, đảm bảo chọn Built-in – đây là SQL Pool được tích hợp sẵn trong không gian làm việc của ta.
Trên thanh công cụ, nhấn nút Run để chạy mã SQL và xem kết quả, có thể giống như sau:
C1	C2	C3	C4
ProductID	ProductName	Category	ListPrice
771	Mountain-100 Silver, 38	Mountain Bikes	3399.99
772	Mountain-100 Silver, 42	Mountain Bikes	3399.99
…	…	…	…
 
Cách khắc phục: Gán quyền đọc cho chính mình(qua Microsoft Entra ID)
1.	Truy cập vào Azure Portal → Vào Storage Account tương ứng.
 


2.	Vào tab Access Control (IAM).


3.	Gán quyền Storage Blob Data Reader cho user của chính mình hoặc managed identity của workspace Synapse.
Bấm nút “+ Add” → “Add role assignment”.
 
Trong hộp thoại hiện ra:
•	Role: chọn Storage Blob Data Reader
 
Next>
•	Assign access to: chọn "User, group or service principal"
 


•	Members: bấm Select members, chọn:


o	Chính mình(nếu chạy query qua tài khoản cá nhân)


o	Hoặc Synapse workspace managed identity nếu đang chạy bằng hệ thống.
 
Select>
 
Bấm Review + Assign.
 
4. Kiểm tra quyền
•	Vào lại Access Control (IAM) > Tab Role assignments


•	Tìm và xác nhận rằng người dùng hoặc Synapse workspace của ta đã có quyền Storage Blob Data Reader
 
Sau khi gán, chờ vài phút → Thử lại câu truy vấn trong Synapse Studio.
 
Lưu ý rằng kết quả có bốn cột C1, C2, C3 và C4, và hàng đầu tiên chứa tên các trường dữ liệu. Để khắc phục vấn đề này, thêm tham số HEADER_ROW = TRUE vào hàm OPENROWSET, ví dụ:
SELECT
    TOP 100 *
FROM
    OPENROWSET(
        BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/product_data/products.csv',
        FORMAT = 'CSV',
        PARSER_VERSION='2.0',
        HEADER_ROW = TRUE
    ) AS [result]
Lúc này kết quả sẽ hiển thị đúng:
ProductID	ProductName	Category	ListPrice
771	Mountain-100 Silver, 38	Mountain Bikes	3399.99
772	Mountain-100 Silver, 42	Mountain Bikes	3399.99
…	…	…	…
 
Tiếp theo, sửa truy vấn như sau để đếm số lượng sản phẩm theo từng danh mục:
SELECT
    Category, COUNT(*) AS ProductCount
FROM
    OPENROWSET(
        BULK 'https://datalakexxxxxxx.dfs.core.windows.net/files/product_data/products.csv',
        FORMAT = 'CSV',
        PARSER_VERSION='2.0',
        HEADER_ROW = TRUE
    ) AS [result]
GROUP BY Category;
Chạy truy vấn sẽ trả về:
Category	ProductCount
Bib Shorts	3
Bike Racks	1
…	…
 
Trong khung Properties của script SQL, đổi tên thành Count Products by Category, rồi nhấn Publish để lưu.
 
 
Đóng tab script, chuyển sang trang Develop, ta sẽ thấy script vừa lưu đã xuất hiện ở đó. Nhấp vào script để mở lại, đảm bảo kết nối với SQL Pool tích hợp sẵn và chạy lại để xem kết quả.
 
Trong khung kết quả, chuyển sang chế độ Chart view, chọn các thiết lập sau:
•	Loại biểu đồ: Column
•	Cột phân loại: Category
•	Cột dữ liệu: ProductCount
•	Vị trí chú thích: bottom - center
Biểu đồ hiển thị sẽ thể hiện số lượng sản phẩm theo từng danh mục.
 
Use a Spark pool to analyze data
Mặc dù SQL là ngôn ngữ phổ biến để truy vấn dữ liệu có cấu trúc, nhưng nhiều nhà phân tích dữ liệu thích dùng Python để khám phá và xử lý dữ liệu. Trong Azure Synapse Analytics, ta có thể chạy mã Python (và các ngôn ngữ khác) trong Spark pool, vốn sử dụng công cụ xử lý dữ liệu phân tán dựa trên Apache Spark.
Trong Synapse Studio, nếu tab chứa products.csv không còn mở, hãy vào trang Data, duyệt đến thư mục product_data, nhấp chuột phải vào products.csv, chọn New notebook rồi Load to DataFrame.
 
Trong Notebook 1, đảm bảo danh sách Attach to đã chọn Spark pool, ví dụ sparkxxxxxxx, và ngôn ngữ là PySpark (Python).
 
Mã trong ô đầu tiên sẽ như sau:
%%pyspark
df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/product_data/products.csv', format='csv'
## If header exists uncomment line below
 ##, header=True
)
display(df.limit(10))
Chạy mã bằng cách nhấn nút Run, lần đầu tiên chạy có thể mất vài phút do Spark pool cần khởi động.
Kết quả sẽ hiện ra như:
c0	c1	c2	c3
ProductID	ProductName	Category	ListPrice
771	Mountain-100 Silver, 38	Mountain Bikes	3399.99
772	Mountain-100 Silver, 42	Mountain Bikes	3399.99
…	…	…	…
 
Giải bỏ dòng , header=True để đọc đúng tiêu đề cột:
%%pyspark
df = spark.read.load('abfss://files@datalakexxxxxxx.dfs.core.windows.net/product_data/products.csv', format='csv'
, header=True
)
display(df.limit(10))
Chạy lại để thấy kết quả đúng:
ProductID	ProductName	Category	ListPrice
771	Mountain-100 Silver, 38	Mountain Bikes	3399.99
772	Mountain-100 Silver, 42	Mountain Bikes	3399.99
…	…	…	…
 
Thêm ô mã mới với nội dung:
df_counts = df.groupby(df.Category).count()
display(df_counts)
Chạy mã để hiển thị số lượng sản phẩm theo từng danh mục:
Category	count
Headsets	3
Wheels	14
…	…
 
Chuyển sang chế độ biểu đồ để xem trực quan.
 
Mở Properties, đổi tên notebook thành Explore products và nhấn Publish để lưu.
 
 
Đóng tab notebook và dừng Spark session khi được hỏi. Kiểm tra trang Develop để đảm bảo notebook đã được lưu.
 
Use a dedicated SQL pool to query a data warehouse.
Do phiên bản Powershell trên Azure portal là phiên bản cũ, không có cài đặt bcp để chạy lệnh load data từ datawarehouse trong github:
 
Và muốn cài đặt bcp phải có quyền Admin trên Powershell ở Azure Portal. Thế nên, cần cài đặt Azure Powershell trên Powershell trên máy local (Windows).

Install Azure PowerShell 
1. Yêu cầu hệ thống
•	Azure PowerShell yêu cầu:
o	Windows: PowerShell 5.1 trở lên.
o	Linux/macOS: PowerShell Core 6.x trở lên (khuyến nghị 6.2.4 hoặc mới hơn).
•	Để kiểm tra phiên bản PowerShell hiện tại, chạy lệnh sau trong PowerShell:
$PSVersionTable.PSVersion 
 
 Đủ yêu cầu phiên bản 5.1 trở lên.
Chạy lệnh sau để kiểm tra đã cài đặt .NET Framework chưa:
(Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP\v4\Full").Release
 
 .NET Framework đã có sẵn
2. Cài đặt PowerShell 5.1 và .NET Framework (nếu cần)
•	Nếu dùng Windows 10 phiên bản 1607 trở lên, đã có sẵn PowerShell 5.1.
•	Nếu chưa, hãy cập nhật Windows hoặc tải và cài đặt PowerShell 5.1 từ Microsoft.
•	Cài đặt .NET Framework 4.7.2 hoặc cao hơn nếu chưa có.
3. Cập nhật PowerShellGet để hỗ trợ lệnh Install-Module:
Chạy lệnh sau để đảm bảo PowerShellGet đang ở phiên bản mới nhất:
Install-Module -Name PowerShellGet -Force
 
4. Cài đặt mô-đun Azure PowerShell
Lưu ý: Không cài đồng thời cả AzureRM và Az trên PowerShell 5.1. Nếu đang dùng AzureRM, hãy gỡ bỏ nó hoặc cài Az bằng PowerShell 6.2.4 trở lên.
Cài đặt cho toàn hệ thống (yêu cầu quyền quản trị)
•	Trên Windows: Mở PowerShell bằng quyền Run as Administrator.
•	Trên Linux/macOS: Sử dụng sudo.
if (Get-Module -Name AzureRM -ListAvailable) {
    Write-Warning -Message ('Az module not installed. Having both the AzureRM and ' +
      'Az modules installed at the same time is not supported.')
} else {
    Install-Module -Name Az -AllowClobber -Scope CurrentUser
}
 
5. Cài đặt ngoại tuyến
Nếu không thể truy cập internet từ máy cài đặt:
•	Trên máy có internet, tải mô-đun bằng lệnh:
Save-Module -Name Az -Path '\\server\share\PowerShell\modules' -Force
•	Sao chép thư mục chứa mô-đun sang máy cần cài đặt và sử dụng Import-Module để sử dụng.

6. Kết nối đến Azure
Để tránh bị lỗi thì:
•	Mở quyền:
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser -Force
•	Nếu ta đã tắt tính năng tự động nạp mô-đun, có thể cần nạp thủ công bằng lệnh :
Import-Module Az.Accounts -Force
 
Sau khi cài đặt xong, chạy lệnh sau để đăng nhập vào Azure:
Connect-AzAccount
 
•	Một cửa sổ trình duyệt sẽ mở ra để ta đăng nhập.
 
Ấn Next và nhập password.
Đã đăng nhập thành công:
Install Azure CLI on Windows
Cài đặt Azure CLI để chạy lệnh az có trong setup.ps1
 
Link hướng dẫn: https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows?pivots=winget
1. Mở PowerShell hoặc Command Prompt
Nên chạy với quyền Administrator để tránh lỗi quyền truy cập.
2. Cài đặt Azure CLI bằng winget (nếu có sẵn)
Nhập lệnh sau trong PowerShell:
winget install --exact --id Microsoft.AzureCLI
 
 

3. Nếu máy chưa có winget, cài đặt bằng MSI
•	Truy cập liên kết sau để tải file cài đặt MSI:
•	https://aka.ms/installazurecliwindows
•	Chạy file .msi vừa tải về và làm theo hướng dẫn cài đặt.
4. Kiểm tra cài đặt
Sau khi cài xong, mở PowerShell/CMD khác, sau đó gõ:
az --version
Nếu thấy thông tin phiên bản, tức là cài đặt thành công.
 

5. Đăng nhập vào Azure
Sử dụng lệnh sau để đăng nhập tài khoản Azure:
az login
Một trình duyệt sẽ mở ra để nhập tài khoản.
 
Continue>
 
Next> và nhập password
 
Nhấn Enter để giá trị mặc định
 
Kiểm tra thông tin subscription đang dùng và phiên bản az:
az account show
az version
 
Clone folder:
git clone https://github.com/MicrosoftLearning/dp-203-azure-data-engineer.git
 
Trên cùng Powershell đó, kết nối AzAccount:
Connect-AzAccount
 
Chọn Continue và thực hiện login:
 
Đăng nhập thành công:
 
 
Chạy lại lệnh chạy file setup.ps1
cd Allfiles/Labs/01
 ./setup.ps1
 
Nhập password
 
 
Thực hiện bài tập:
 
 
Mở Synapse Studio và thực hiện các yêu cầu
Các dữ liệu file-based từ data lake thường được xử lý rồi tải vào kho dữ liệu quan hệ để phục vụ nhu cầu BI. Trong Synapse Analytics, kho dữ liệu có thể triển khai bằng dedicated SQL pool.
Trong Synapse Studio, vào trang Manage, tại phần SQL pools, chọn hàng sqlxxxxxxx và nhấn nút Resume.
 
Chờ cho pool khởi động xong (trạng thái sẽ là Online). 
 
Vào trang Data, chuyển sang tab Workspace, mở phần SQL databases, tìm cơ sở dữ liệu sqlxxxxxxx.
 
Mở thư mục Tables, tại bảng FactInternetSales, chọn New SQL script > Select TOP 100 rows.
 
Kết quả sẽ là 100 giao dịch bán hàng đầu tiên đã được thiết lập sẵn trong quá trình cài đặt môi trường.
 
 
Thay thế truy vấn bằng đoạn mã sau:
SELECT d.CalendarYear, d.MonthNumberOfYear, d.EnglishMonthName,
       p.EnglishProductName AS Product, SUM(o.OrderQuantity) AS UnitsSold
FROM dbo.FactInternetSales AS o
JOIN dbo.DimDate AS d ON o.OrderDateKey = d.DateKey
JOIN dbo.DimProduct AS p ON o.ProductKey = p.ProductKey
GROUP BY d.CalendarYear, d.MonthNumberOfYear, d.EnglishMonthName, p.EnglishProductName
ORDER BY d.MonthNumberOfYear
Chạy mã để xem số lượng sản phẩm bán ra theo tháng và năm.
 

Lưu lại file:
 
 
Vào develop để kiểm tra:
 
Vào Manage và nhấn Pause để dừng sqlxxxx nhằm tiết kiệm tài nguyên:
 
 
 
Xuất CSV:
Thành công:
 
Mở Properties, đặt tên truy vấn là Aggregate product sales, nhấn Publish để lưu.
 
 

Explore data with a Data Explorer pool
1. Giới thiệu
Azure Synapse Data Explorer cung cấp một môi trường thực thi (runtime) cho phép lưu trữ và truy vấn dữ liệu bằng cách sử dụng Kusto Query Language (KQL). Ngôn ngữ Kusto được tối ưu hóa cho dữ liệu có thành phần theo chuỗi thời gian, ví dụ như dữ liệu thời gian thực từ tệp nhật ký (log files) hoặc thiết bị IoT.
Tạo một cơ sở dữ liệu Data Explorer và ingest dữ liệu vào bảng
Bước 1: Mở và khởi động Data Explorer pool
•	Trong Synapse Studio, vào trang Manage.
•	Trong mục Data Explorer pools, chọn hàng chứa pool có tên dạng adxxxxxxxx, sau đó chọn Resume để khởi động pool.
•	Chờ cho đến khi pool được khởi động hoàn tất. Việc này có thể mất một khoảng thời gian. Ta có thể nhấn nút Refresh để kiểm tra trạng thái. Khi trạng thái hiển thị là Online, pool đã sẵn sàng.
Bước 2: Tạo cơ sở dữ liệu sales-data
•	Chuyển đến trang Data.
•	Trong tab Workspace, mở rộng phần Data Explorer Databases và kiểm tra xem pool adxxxxxxxx có được liệt kê hay không. Nếu chưa, nhấn Refresh để làm mới.
•	Trong khung Data, chọn Create new Data Explorer database, chọn pool adxxxxxxxx, nhập tên là sales-data và tạo cơ sở dữ liệu.
•	Chờ thông báo xác nhận rằng cơ sở dữ liệu đã được tạo.
Bước 3: Ingest dữ liệu vào bảng
•	Chuyển sang trang Develop, trong danh sách KQL scripts, chọn ingest-data.
•	Script này gồm hai lệnh:
o	Lệnh .create table để tạo bảng tên là sales.
o	Lệnh .ingest into table để nạp dữ liệu vào bảng từ nguồn HTTP.
•	Trong phần cấu hình script:
o	Chọn pool là adxxxxxxxx.
o	Chọn cơ sở dữ liệu là sales-data.
•	Thực hiện các bước sau:
o	Bôi đen lệnh .create table, sau đó chọn Run để tạo bảng sales.
o	Tiếp theo, bôi đen lệnh .ingest into table và chạy để nạp dữ liệu vào bảng.
Lưu ý: Trong ví dụ này, ta chỉ nạp một lượng nhỏ dữ liệu dạng batch từ một tệp tin. Trong thực tế, Azure Data Explorer có thể xử lý khối lượng dữ liệu lớn hơn rất nhiều, bao gồm cả dữ liệu thời gian thực từ các nguồn như Azure Event Hubs.
Sử dụng Kusto Query Language để truy vấn bảng dữ liệu
Bước 1: Mở bảng và kiểm tra dữ liệu
•	Trở lại trang Data.
•	Trong menu của cơ sở dữ liệu sales-data, chọn Refresh để cập nhật.
•	Mở rộng thư mục Tables, tìm bảng sales.
•	Trong menu bảng sales, chọn New KQL script > Take 1000 rows.
Script được tạo ra sẽ có dạng sau:
sales
| take 1000
•	Chạy truy vấn để xem kết quả – đây là 1000 dòng dữ liệu đầu tiên trong bảng.
Bước 2: Thực hiện truy vấn lọc dữ liệu
Truy vấn theo tên sản phẩm:
sales
| where Item == 'Road-250 Black, 48'
•	Chạy truy vấn để chỉ lấy các dòng có sản phẩm là Road-250 Black, 48.
Truy vấn theo năm:
sales
| where Item == 'Road-250 Black, 48'
| where datetime_part('year', OrderDate) > 2020
•	Truy vấn này lọc thêm điều kiện là các đơn hàng sau năm 2020.
Truy vấn doanh thu theo năm:
sales
| where OrderDate between (datetime(2020-01-01 00:00:00) .. datetime(2020-12-31 23:59:59))
| summarize TotalNetRevenue = sum(UnitPrice) by Item
| sort by Item asc
•	Truy vấn này tổng hợp doanh thu thuần (TotalNetRevenue) theo từng sản phẩm trong năm 2020 và sắp xếp theo tên sản phẩm.
Bước 3: Lưu và xuất bản truy vấn
•	Nếu khung Properties chưa hiển thị, hãy mở nó.
•	Đặt tên truy vấn là Explore sales data.
•	Chọn Publish để lưu truy vấn.
•	Đóng tab truy vấn và quay lại trang Develop để xác nhận rằng script đã được lưu.
Tạm dừng pool và dọn dẹp tài nguyên
Tạm dừng Data Explorer pool:
•	Trở lại trang Manage.
•	Chọn hàng chứa Data Explorer pool adxxxxxxxx và chọn Pause để tạm dừng pool.
Delete Azure resources
•	Đóng tab Synapse Studio và quay lại Azure portal.
•	Trên trang chính, chọn Resource groups.
•	Chọn nhóm tài nguyên có tên dạng dp000-xxxxxxx (không chọn nhóm được quản lý).
•	Xác nhận rằng nhóm tài nguyên này chứa workspace Synapse, storage account, SQL pool, Data Explorer pool và Spark pool.
•	Trên trang tổng quan của nhóm tài nguyên, chọn Delete resource group.
•	Nhập tên nhóm tài nguyên để xác nhận và chọn Delete.
Sau vài phút, toàn bộ nhóm tài nguyên và workspace liên quan sẽ bị xóa để tránh phát sinh chi phí không cần thiết.
 

