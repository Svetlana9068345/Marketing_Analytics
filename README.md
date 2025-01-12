# Marketing-Analytics
## Постановка задачи
Эта информационная панель помогает компании:
- выделить ключевые этапы воронки, на которых посетители уходят;
- проследить, как изменяется конверсия в течение года для каждого продукта;
- увидеть, существует ли взаимосвязь между коэффициентом конверсии и оценкой продукта;
- определить, какие типы контента вызывают наибольшую вовлеченность у клиентов;
- проследить, какие оценки клиенты ставят чаще всего;
- провести анализ тональности отзывов и проверить, соответствуют ли оценки клиентов комментариям, которые они оставляют.
## Панели дашборда
![Overview](https://github.com/user-attachments/assets/58096d36-82ff-4680-bf33-8ba75fbc1975)
![Conversion Details](https://github.com/user-attachments/assets/3920ed64-7a9e-4db4-b9e4-265b38bf1c6f)
![Social Media Details](https://github.com/user-attachments/assets/bb8c22ee-c632-4430-88e5-dead95238803)
![Customer Review Details](https://github.com/user-attachments/assets/7607d0e7-c942-447f-8f3b-6de6ff76d5f2)
## Стратегии по улучшению после проведенного анализа 
### Увеличение коэффициента конверсии:
- Необходимо сосредоточить маркетинговые усилия на продуктах с продемонстрированным высоким коэффициентом конверсии (например,  Kayaks, Ski Boots, and Baseball Gloves). 
- Внедряйте сезонные акции или персонализированные кампании в пиковые месяцы (например, январь и сентябрь), чтобы извлечь выгоду из этих тенденций.
### Улучшение взаимодействие с клиентами:
- Чтобы изменить ситуацию с падением просмотров и низким уровнем взаимодействия, поэкспериментируйте с более привлекательными форматами контента, такими как интерактивные видео или пользовательский контент. 
- Кроме того, повысьте вовлеченность, разместив призывы к действию в социальных сетях, особенно в месяцы с исторически низким уровнем вовлеченности (сентябрь-декабрь).
### Улучшение показателей обратной связи с клиентами:
- Внедрите цикл обратной связи, в котором смешанные и отрицательные отзывы (Mixed Positive,  Mixed Negative,  Negative) анализируются для выявления общих проблем. 
- Рассмотрите возможность взаимодействия с недовольными клиентами для решения проблем.

### Выполненные шаги
- Шаг 1 : Восстановление резервной копии базы данных в MMSM (SQL Server Management Studio).
- Шаг 2 : Расширенный анализ настроений с помощью Python.
- Шаг 3 : Импорт данных в Power BI из базы данных с применением SQL-запросов для очистки и подготовки данных.
- Шаг 4 : Преобразование данных в Power Query.
- Шаг 5 : Построение реляционной модели данных.
- Шаг 6 : Создание мер DAX.
- Шаг 7 : Создание интерактивного отчета для анализа и визуализации данных.
- Шаг 8 : Формирование стратегий по улучшению коэффициента конверсии, взаимодействия с клиентами, показателей обратной связи после проведенного анализа.
### Меры DAX
-     Views = SUM ( fact_engagement_data[Views] )
-     Clicks = SUM (fact_engagement_data[Clicks])
-      Likes = SUM ( fact_engagement_data[Likes] )
-     Conversion Rate = 
VAR TotalVisitors = CALCULATE( COUNT (fact_customer_journey[JourneyID]) , fact_customer_journey[Action] = "View" )
VAR TotalPurchases = CALCULATE(
    COUNT(fact_customer_journey[JourneyID]),
    fact_customer_journey[Action] = "Purchase"
)
RETURN
IF(
    TotalVisitors = 0, 
    0, 
    DIVIDE(TotalPurchases, TotalVisitors)
)
-     Number of Campaign Engagements = DISTINCTCOUNT( fact_engagement_data[EngagementID] )
-     Number of Campaigns = DISTINCTCOUNT( fact_engagement_data[CampaignID] )
-     Number of Customer Journeys = DISTINCTCOUNT( fact_customer_journey[JourneyID] )
-     Number of Customer Reviews = DISTINCTCOUNT( fact_customer_reviews_with_sentiment[ReviewID] )
-     Rating (Average) = AVERAGE( fact_customer_reviews_with_sentiment[Rating] )
### SQL-запросы к базе данных
## SQL-запрос к таблице dim_customers
    SELECT 
    c.CustomerID,  
    c.CustomerName,  
    c.Email,  
    c.Gender,  
    c.Age,  
    g.Country,  data
    g.City  
    FROM 
    dbo.customers as c  
    dbo.geography g  
    ON 
    c.GeographyID = g.GeographyID
## SQL-запрос к таблице fact_customer_journey

    SELECT 
    JourneyID,  
    CustomerID,   
    ProductID,   
    VisitDate,  
    Stage,  
    Action,  
    COALESCE(Duration, avg_duration) AS Duration 
    FROM 
        (SELECT 
            JourneyID,  
            CustomerID,  
            ProductID,  
            VisitDate,  
            UPPER(Stage) AS Stage,  
            Action,  
            Duration,  
            AVG(Duration) OVER (PARTITION BY VisitDate) AS avg_duration,  
            ROW_NUMBER() OVER (
                PARTITION BY CustomerID, ProductID, VisitDate, UPPER(Stage), Action 
                ORDER BY JourneyID  
            ) AS row_num  
            FROM 
            dbo.customer_journey  
    ) AS subquery  
    WHERE 
    row_num = 1
## SQL-запрос к таблице fact_customer_reviews
    SELECT 
    ReviewID, 
    CustomerID,  
    ProductID,  
    ReviewDate,  
    Rating,   
    REPLACE(ReviewText, '  ', ' ') AS ReviewText
    FROM 
    dbo.customer_reviews  
## SQL-запрос к таблице fact_engagement_data
    SELECT 
    EngagementID, 
    ContentID,  
    CampaignID,  
    ProductID,  
    UPPER(REPLACE(ContentType, 'Socialmedia', 'Social Media')) AS ContentType,  
    LEFT(ViewsClicksCombined, CHARINDEX('-', ViewsClicksCombined) - 1) AS Views,  
    RIGHT(ViewsClicksCombined, LEN(ViewsClicksCombined) - CHARINDEX('-', ViewsClicksCombined)) AS Clicks,  
    Likes, 
    FORMAT(CONVERT(DATE, EngagementDate), 'dd.MM.yyyy') AS EngagementDate  
    FROM 
    dbo.engagement_data  
    WHERE 
    ContentType != 'Newsletter'
## SQL-запрос к таблице dim_products
    SELECT 
    ProductID,  
    ProductName,  
    Price,  
    CASE 
        WHEN Price < 50 THEN 'Low' 
        WHEN Price BETWEEN 50 AND 200 THEN 'Medium'  
        ELSE 'High'  
    END AS PriceCategory  

    FROM 
    dbo.products
