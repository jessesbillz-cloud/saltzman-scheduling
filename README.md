[README.md](https://github.com/user-attachments/files/25409070/README.md)
# DSA Inspection Scheduling System

A web-based scheduling platform for California Division of the State Architect (DSA) certified inspectors and construction teams.

## Overview

This system streamlines the inspection request process for DSA projects by providing:
- **Contractor-friendly scheduling interface** with visual calendar
- **Automated notifications** via email and push alerts
- **Real-time calendar integration** with ICS feed support
- **Project isolation** ensuring team privacy
- **Mobile-optimized design** for field use

## Live Forms

- **WPMS**: [Woodland Park Middle School](https://jessesbillz-cloud.github.io/saltzman-scheduling/) (Lusardi)
- **CSUSM**: [CSU San Marcos](https://jessesbillz-cloud.github.io/saltzman-scheduling/csusm.html) (CW Driver)  
- **Demo**: [Try the system](https://jessesbillz-cloud.github.io/saltzman-scheduling/demo.html) (Other DSA inspectors)

## Features

### For Contractors
- **Visual Calendar**: See available slots and existing inspections
- **Smart Conflict Detection**: Prevents double-booking for Regular/IOR inspections  
- **Flexible Scheduling**: "Flexible" option for same-day scheduling with noon reminders
- **Special Inspection Types**: Material ID, CWI, Pull Test, Bolting, Soils, Concrete, Fireproofing
- **Duration Selection**: 30 min, 1 hr, 2 hr, 4 hr, All Day
- **Photo/PDF Upload**: Attach inspection requests directly
- **Team Notifications**: CC relevant project team members
- **Edit/Cancel**: Modify or cancel submitted requests with audit trail

### For Inspectors
- **Calendar Sync**: ICS feed integrates with iPhone/Google Calendar
- **Push Notifications**: Real-time alerts via Ntfy
- **Email Notifications**: Professional email alerts with PDF attachments
- **Project Separation**: Complete isolation between different projects
- **Audit Logging**: Track all changes and cancellations
- **Conflict Management**: Smart scheduling prevents overlaps

## Technical Architecture

### Frontend
- **Pure HTML/CSS/JavaScript** - No frameworks, optimized for speed
- **Dark theme** with project-specific accent colors
- **PWA-ready** for mobile installation
- **Responsive design** for field use on phones/tablets

### Backend
- **Supabase** - PostgreSQL database with real-time capabilities
- **Edge Functions** - Serverless request processing
- **Resend** - Professional email delivery
- **Ntfy** - Push notification system
- **GitHub Pages** - Static hosting

### Database Schema
```sql
inspection_requests (
  id uuid PRIMARY KEY,
  project text,
  inspection_date date,
  inspection_time time,
  inspection_types jsonb,
  special_type text,
  duration text,
  flexible boolean,
  flexible_display text,
  submitted_by text,
  notes text,
  status text DEFAULT 'active',
  created_at timestamptz
)

inspection_log (
  id uuid PRIMARY KEY,
  request_id uuid REFERENCES inspection_requests(id),
  action text,
  action_by text,
  details text,
  created_at timestamptz
)
```

## Project Isolation

Each project operates independently:
- **WPMS Team**: Ron Davis, Scuyler Lindgren, Alejandro Jimenez (Lusardi/SOCOTEC)
- **CSUSM Team**: Kyle Haider, Mike McConkey, Brayden Duzan, Matt Lulling (CW Driver/OCMI)

Teams cannot see:
- Other project's Special inspections on calendar
- Other project's team members in edit/cancel modals
- Other project's email notifications

## Business Model

### Current: Freemium Lead Generation
- **Free Tier**: Inspection scheduling system (this repository)
- **Paid Tier**: [My Daily Reports](https://mydailyreports.com) - $19.99/month
  - Voice-to-document automation with Ray-Ban glasses
  - Auto-generate DSA forms (151, 155, 156)
  - AI-powered report completion
  - Document workflow automation

### Target Market
- 800+ DSA certified inspectors in California
- Construction GCs managing state projects
- Special inspection companies (CWI, soils, concrete, etc.)

## Setup Instructions

### 1. Database Setup
```sql
-- Create tables in Supabase
CREATE TABLE inspection_requests (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  project text NOT NULL,
  inspection_date date NOT NULL,
  inspection_time time NOT NULL,
  inspection_types jsonb DEFAULT '[]',
  special_type text,
  duration text,
  flexible boolean DEFAULT false,
  flexible_display text,
  gc text,
  subcontractor text DEFAULT '',
  location_detail text DEFAULT '',
  submitted_by text,
  notes text DEFAULT '',
  status text DEFAULT 'active',
  created_at timestamptz DEFAULT now()
);

CREATE TABLE inspection_log (
  id uuid DEFAULT gen_random_uuid() PRIMARY KEY,
  request_id uuid REFERENCES inspection_requests(id),
  action text NOT NULL,
  action_by text NOT NULL,
  details text DEFAULT '',
  created_at timestamptz DEFAULT now()
);
```

### 2. Environment Variables
```bash
# Supabase
SUPABASE_URL=your-project-url
SUPABASE_SERVICE_ROLE_KEY=your-service-key

# Email (Resend)
RESEND_API_KEY=your-resend-key

# Notifications (Ntfy)
NTFY_TOPIC=your-topic-name
```

### 3. Deploy Edge Functions
Deploy `submit-inspection` and `update-inspection` functions to Supabase.

### 4. Configure Notifications
- **Email**: Verify domain in Resend
- **Push**: Set up Ntfy topic for `saltzman-vis-alerts`

## API Endpoints

### Submit Inspection
```
POST /functions/v1/submit-inspection
Content-Type: multipart/form-data

{
  photo: File,
  project: string,
  inspection_date: string,
  inspection_time: string,
  inspection_types: string[], // JSON
  special_type?: string,
  duration: string,
  flexible?: boolean,
  submitted_by: string,
  notes?: string,
  email_recipients?: string[], // JSON
  skip_conflict?: boolean
}
```

### Update Inspection  
```
POST /functions/v1/update-inspection
Content-Type: application/json

{
  request_id: string,
  action: "edit" | "cancel",
  action_by: string,
  new_date?: string,
  new_time?: string,
  reason?: string
}
```

## Contributing

This is a production system serving active construction projects. Changes should be:
1. Tested on demo form first
2. Backward compatible with existing data
3. Coordinated with project teams for any UI changes

## License

Private repository - Saltzman Inspections proprietary system.

## Contact

**Jesse Saltzman** - DSA Certified Inspector #6160  
**Saltzman Inspections** - CalVet Micro Business  
**Email**: saltzman.jesse@gmail.com  

---

*Part of the Inspector Bot ecosystem - automating DSA inspection workflows for California's construction industry.*
