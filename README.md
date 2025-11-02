# Maven Fuzzy Factory E-Commerce Analytics

## 📊 Project Overview

This project analyzes and optimizes the online retail performance of Maven Fuzzy Factory, a teddy bear e-commerce store. It leverages detailed marketing, website session, order, and product data to drive actionable business insights.

## 🎯 Key Features

- **Session-to-Order Conversion Tracking**: Monitor conversion rates across marketing channels
- **Revenue Performance Analysis**: Analyze revenue per order and per session
- **User Behavior Insights**: Understand customer patterns and preferences
- **Product Performance**: Track product popularity and returns analysis
- **Marketing Channel Effectiveness**: Evaluate which channels drive the best results

## 🛠️ Technology Stack

- **Microsoft Fabric Lakehouse**: Data storage and management
- **MS Fabric Data Pipelines**: ETL workflows and transformations
- **Power BI**: Interactive dashboards and business intelligence
- **GitHub**: Version control and collaboration
- **GitHub Actions**: CI/CD automation

## 📁 Project Structure

├── docs/ # Documentation and project context
│ ├── master-context.md
│ ├── project-overview.md
│ └── workflow-architecture.md
├── data/ # Data files and data dictionary
│ └── data-dictionary.csv
├── scripts/ # ETL and transformation scripts
├── pipelines/ # Data pipeline configurations
├── dashboards/ # Power BI dashboard files
├── governance/ # Data governance guidelines
└── README.md # This file


## 📚 Documentation

For detailed information, see:
- **Master Context**: `docs/master-context.md` - Core objectives and project standards
- **Project Overview**: `docs/project-overview.md` - Detailed project goals and features
- **Workflow & Architecture**: `docs/workflow-architecture.md` - Technical architecture and data flow
- **Data Dictionary**: `data/data-dictionary.csv` - Data schema and field definitions

## 🔄 Project Workflow

1. **Data Ingestion**: Raw CSV files uploaded to MS Fabric Lakehouse staging area
2. **ETL Processing**: Data cleaned, deduplicated, and enriched using MS Fabric Dataflows
3. **Data Loading**: Processed data loaded into curated Lakehouse tables
4. **Analytics & Visualization**: Power BI dashboards provide business insights
5. **Automation**: GitHub Actions schedule pipeline refresh and deployment

## 📊 Dataset Information

### Data Source

This project uses the **Toy Store E-Commerce Database** from Maven Analytics:
- **Dataset**: [Free Sample Dataset Download - Toy Store E-Commerce Database](https://mavenanalytics.io/data-playground/toy-store-e-commerce-database)
- **License**: Public Domain
- **Provider**: Maven Analytics

### Dataset Overview

The Maven Fuzzy Factory e-commerce database includes:
- **Website Sessions & Pageviews**: Detailed tracking by user
- **Order Data**: Complete transaction records
- **Product Information**: Catalog and product details
- **Returns & Refunds**: Customer return patterns
- **Marketing Data**: Channel performance and campaign details

### Recommended Analysis Areas

- Trend analysis in website sessions and order volume
- Session-to-order conversion rate trends
- Marketing channel performance evaluation
- Revenue per order and revenue per session evolution
- Impact analysis of new product launches

## 📊 Current Status

- **Phase**: Data exploration and pipeline design
- **Upcoming**: Data ingestion setup → ETL pipeline creation → Dashboard development

## 👥 Access & Contribution

This repository is **public and read-only**. 

- **Viewers**: Can view all documentation, code, and project structure
- **Contributions**: To suggest changes or improvements, please open a Pull Request

## 📖 How to Use This Repository

1. Read the documentation in `/docs` to understand the project scope and architecture
2. Review the data dictionary in `/data` for data schema information
3. Check pipeline configurations in `/pipelines` for ETL workflows
4. Explore Power BI dashboards in `/dashboards` for business insights

## 🔗 Related Links

- [Microsoft Fabric Documentation](https://learn.microsoft.com/en-us/fabric/)
- [Power BI Learning Resources](https://learn.microsoft.com/en-us/power-bi/)
- [GitHub Best Practices](https://docs.github.com/en)

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.

---

**Last Updated**: November 2, 2025
