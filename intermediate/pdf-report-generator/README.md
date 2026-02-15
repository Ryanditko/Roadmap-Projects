# PDF Report Generator

**Level:** Intermediate | **Time:** 1 week

## Description
Generate professional PDF reports with charts, tables, images, and custom styling. Learn PDF generation, data visualization, and document formatting.

## Core Features
- Dynamic PDF generation
- Charts and graphs (Chart.js)
- Tables with styling
- Images and logos
- Custom headers/footers
- Page numbering
- Export functionality

## Implementation Guide

```javascript
class PDFReportGenerator {
    constructor() {
        this.doc = null;
    }
    
    async generateReport(data) {
        // Using jsPDF library
        const { jsPDF } = window.jspdf;
        this.doc = new jsPDF();
        
        // Title
        this.addTitle(data.title);
        
        // Add content sections
        let yPosition = 30;
        
        for (const section of data.sections) {
            if (section.type === 'text') {
                yPosition = this.addText(section.content, yPosition);
            } else if (section.type === 'table') {
                yPosition = this.addTable(section.data, yPosition);
            } else if (section.type === 'chart') {
                yPosition = await this.addChart(section.data, yPosition);
            }
        }
        
        // Add footer
        this.addFooter();
        
        return this.doc;
    }
    
    addTitle(title) {
        this.doc.setFontSize(24);
        this.doc.setTextColor(40, 40, 40);
        this.doc.text(title, 105, 20, { align: 'center' });
        
        // Underline
        this.doc.setLineWidth(0.5);
        this.doc.line(20, 25, 190, 25);
    }
    
    addText(text, yPosition) {
        this.doc.setFontSize(12);
        this.doc.setTextColor(60, 60, 60);
        
        const lines = this.doc.splitTextToSize(text, 170);
        this.doc.text(lines, 20, yPosition);
        
        return yPosition + (lines.length * 7) + 10;
    }
    
    addTable(data, yPosition) {
        this.doc.autoTable({
            startY: yPosition,
            head: [data.headers],
            body: data.rows,
            theme: 'grid',
            styles: {
                fontSize: 10,
                cellPadding: 5
            },
            headStyles: {
                fillColor: [41, 128, 185],
                textColor: 255,
                fontStyle: 'bold'
            }
        });
        
        return this.doc.lastAutoTable.finalY + 15;
    }
    
    async addChart(data, yPosition) {
        // Create temporary canvas for chart
        const canvas = document.createElement('canvas');
        canvas.width = 600;
        canvas.height = 400;
        
        const ctx = canvas.getContext('2d');
        
        // Generate chart using Chart.js
        const chart = new Chart(ctx, {
            type: data.type,
            data: {
                labels: data.labels,
                datasets: [{
                    label: data.label,
                    data: data.values,
                    backgroundColor: 'rgba(54, 162, 235, 0.5)',
                    borderColor: 'rgba(54, 162, 235, 1)',
                    borderWidth: 2
                }]
            },
            options: {
                responsive: false,
                animation: false
            }
        });
        
        // Wait for chart to render
        await new Promise(resolve => setTimeout(resolve, 100));
        
        // Convert chart to image
        const imgData = canvas.toDataURL('image/png');
        
        // Add to PDF
        this.doc.addImage(imgData, 'PNG', 20, yPosition, 170, 100);
        
        // Cleanup
        chart.destroy();
        
        return yPosition + 110;
    }
    
    addFooter() {
        const pageCount = this.doc.internal.getNumberOfPages();
        
        for (let i = 1; i <= pageCount; i++) {
            this.doc.setPage(i);
            this.doc.setFontSize(10);
            this.doc.setTextColor(150);
            
            const date = new Date().toLocaleDateString();
            this.doc.text(`Generated on ${date}`, 20, 285);
            this.doc.text(`Page ${i} of ${pageCount}`, 190, 285, { align: 'right' });
        }
    }
    
    save(filename) {
        this.doc.save(filename);
    }
    
    preview() {
        const blob = this.doc.output('blob');
        const url = URL.createObjectURL(blob);
        window.open(url);
    }
}

// Example usage
async function generateSalesReport() {
    const generator = new PDFReportGenerator();
    
    const reportData = {
        title: 'Sales Report Q4 2024',
        sections: [
            {
                type: 'text',
                content: 'This report summarizes sales performance for Q4 2024.'
            },
            {
                type: 'table',
                data: {
                    headers: ['Month', 'Revenue', 'Units Sold', 'Growth'],
                    rows: [
                        ['October', '$45,000', '350', '+12%'],
                        ['November', '$52,000', '410', '+15%'],
                        ['December', '$68,000', '530', '+31%']
                    ]
                }
            },
            {
                type: 'chart',
                data: {
                    type: 'bar',
                    label: 'Monthly Revenue',
                    labels: ['Oct', 'Nov', 'Dec'],
                    values: [45000, 52000, 68000]
                }
            }
        ]
    };
    
    await generator.generateReport(reportData);
    generator.save('sales-report.pdf');
}
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>PDF Report Generator</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.31/jspdf.plugin.autotable.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 50px auto;
            padding: 20px;
        }
        .form-group {
            margin-bottom: 20px;
        }
        label {
            display: block;
            margin-bottom: 5px;
            font-weight: 600;
        }
        input, textarea, select {
            width: 100%;
            padding: 10px;
            border: 2px solid #ddd;
            border-radius: 5px;
        }
        button {
            background: #3498db;
            color: white;
            padding: 12px 24px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
        }
    </style>
</head>
<body>
    <h1>PDF Report Generator</h1>
    
    <div class="form-group">
        <label>Report Title</label>
        <input type="text" id="reportTitle" placeholder="Enter report title">
    </div>
    
    <div class="form-group">
        <label>Content</label>
        <textarea id="reportContent" rows="6" placeholder="Enter report content"></textarea>
    </div>
    
    <button onclick="generateSalesReport()">Generate Report</button>
    <button onclick="generator.preview()">Preview</button>
</body>
</html>
```

## Technology Stack
- jsPDF for PDF generation
- jsPDF-AutoTable for tables
- Chart.js for visualizations
- Canvas API

## Testing Checklist
- [ ] PDF generates correctly
- [ ] Tables render properly
- [ ] Charts display in PDF
- [ ] Images embed correctly
- [ ] Page numbers work
- [ ] Footer appears on all pages

## License
MIT - [Coding Projects Library](https://github.com/Ryanditko/coding-projects-library)
