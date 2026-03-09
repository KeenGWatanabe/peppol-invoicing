**Absolutely!** Creating a Next.js app to connect to an Access Point Provider's REST API is a very practical and efficient approach. This is exactly how many modern e-invoicing solutions are built today.

## ✅ Yes, This Approach Works Perfectly

You can build a Next.js application that:
- Takes user input (or uploaded Excel/PDF files)
- Generates the required XML in the background (or lets the API handle it)
- Sends the invoice via the Access Point's API
- Receives delivery status and responses

## 🏗️ Recommended Architecture

```typescript
// Your Next.js App (Frontend + Backend)
// ├── pages/api/* - Your API routes that handle authentication and API calls
// ├── components/* - UI for uploading invoices, tracking status
// └── lib/* - Helpers for XML generation, API client for Access Point
```

## 🔌 Finding Access Point Providers with REST APIs

Here are some IMDA-accredited Access Point Providers that offer developer-friendly APIs:

| Provider | API Documentation | Notes |
|----------|-------------------|-------|
| **Edentri** | Request access via their website | Singapore-based, offers PDF/Excel to Peppol conversion services |
| **Esker** | Developer portal available | Enterprise-focused but has APIs |
| **SAP** | SAP Business Network APIs | For enterprise integration |
| **TradeGecko** | (Now QuickBooks Commerce) | Has API access |
| **Einvoice Network** | einvoice.com.sg | Singapore provider with API options |

## 💻 Implementation Example for Next.js

Here's a conceptual implementation to get you started:

### 1. **API Route to Submit Invoice** (`pages/api/submit-invoice.ts`)

```typescript
// pages/api/submit-invoice.ts
import { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ message: 'Method not allowed' });
  }

  try {
    const { invoiceData } = req.body;

    // 1. Validate the invoice data against PINT-SG schema
    // 2. Generate XML (or send JSON if your Access Point accepts it)
    // 3. Send to Access Point Provider's API
    const response = await fetch('https://api.accesspoint.com/v1/invoices', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${process.env.ACCESS_POINT_API_KEY}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        // Format this according to your Access Point's API spec
        invoice: invoiceData,
        recipient: {
          uen: req.body.uen, // Your customer's UEN
        },
        format: 'PINT-SG',
      }),
    });

    const data = await response.json();
    
    // 4. Return the tracking ID/status to your frontend
    return res.status(200).json({
      success: true,
      trackingId: data.trackingId,
      status: data.status,
    });
    
  } catch (error) {
    console.error('Invoice submission failed:', error);
    return res.status(500).json({ 
      success: false, 
      error: 'Failed to submit invoice' 
    });
  }
}
```

### 2. **Frontend Component** (`components/InvoiceUploader.tsx`)

```typescript
// components/InvoiceUploader.tsx
import { useState } from 'react';

export default function InvoiceUploader() {
  const [file, setFile] = useState<File | null>(null);
  const [status, setStatus] = useState('');

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    // Read file, extract data, or send file to your API
    // which will handle the conversion
    
    const formData = new FormData();
    formData.append('invoice', file!);
    
    const response = await fetch('/api/submit-invoice', {
      method: 'POST',
      body: formData,
    });
    
    const result = await response.json();
    setStatus(`Invoice submitted! Tracking ID: ${result.trackingId}`);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input 
        type="file" 
        accept=".pdf,.xlsx,.xls"
        onChange={(e) => setFile(e.target.files?.[0] || null)}
      />
      <button type="submit">Send to InvoiceNow</button>
      {status && <p>{status}</p>}
    </form>
  );
}
```

## 📦 Key Libraries You'll Need

```bash
npm install xml2js # For XML generation/parsing
npm install zod # For validating invoice data against schema
npm install pdf-parse # If extracting data from PDFs
npm install xlsx # For Excel file parsing
```

## 🧩 The Data Flow

1. **User uploads** Excel/PDF in your Next.js frontend
2. **Your backend** (Next.js API routes) processes the file
   - Extracts structured data from Excel (easy with `xlsx` library)
   - Or uses OCR/parsing for PDFs (more complex)
3. **Validate data** against PINT-SG requirements
4. **Transform to XML** (or the format your Access Point API expects)
5. **Send to Access Point API** with your credentials
6. **Store tracking ID** and show status to user
7. **Handle webhooks** from Access Point for delivery confirmations

## 🎯 Practical Tips for Your Development

1. **Start with a simple JSON API** - Many Access Points accept JSON and convert to Peppol XML themselves. This is easier than generating XML yourself initially.

2. **Request sandbox access** - All providers offer test environments. Get these credentials first.

3. **Focus on the UEN validation** - The recipient's UEN (Unique Entity Number) is critical. Your app needs to validate this before submission.

4. **Handle responses asynchronously** - InvoiceNow delivery can take seconds to minutes. Use webhooks or polling to get final status.

5. **Consider file processing libraries**:
   - **Excel**: `xlsx` or `exceljs` for reading Excel files
   - **PDF**: `pdf2json` or `pdf-parse` for text extraction

## 🚀 Getting Started Path

1. **Week 1**: Contact 2-3 Access Point Providers, get sandbox API keys
2. **Week 2**: Build basic Next.js API integration with their test endpoints
3. **Week 3**: Add Excel parsing and validation
4. **Week 4**: Build UI and handle PDF uploads
5. **Week 5**: Test with real invoices in sandbox environment

## ⚠️ Important Considerations

- **Accreditation**: If you sell this to other businesses, you may need IMDA accreditation as a Solution Provider
- **Data privacy**: Invoice data is sensitive - ensure proper encryption and security
- **GST validation**: Singapore has specific GST rules you must validate
- **Archive requirements**: Invoices must be stored for 5 years (consider using cloud storage)

Would you like me to help you find specific Access Point Providers with the most developer-friendly APIs, or would you prefer more detailed code examples for parsing Excel/PDF files in Next.js?
