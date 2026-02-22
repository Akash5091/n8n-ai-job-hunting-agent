# Email Templates

Customizable email templates for the n8n AI Job Hunting Agent. Modify these templates to match your branding and communication style.

## Template Variables

Common variables available in all templates:
- `{{candidate_name}}` - Full name of the candidate
- `{{job_title}}` - Title of the position
- `{{company_name}}` - Name of the company
- `{{your_name}}` - Sender's name
- `{{your_email}}` - Sender's email address
- `{{date}}` - Current date
- `{{ats_score}}` - ATS matching score

## 1. Initial Outreach Email

**Subject:** Software Engineer at {{company_name}} - Your expertise matches our needs

```html
Dear Hiring Manager,

I am writing to express my strong interest in the {{job_title}} position at {{company_name}}. 
With my proven track record in full-stack development and expertise in the technologies you seek, 
I am confident that I can make a significant contribution to your team.

My qualifications include:
- X years of professional software development experience
- Proficiency in Java, JavaScript, TypeScript, Python, and Go
- Strong background in microservices architecture and cloud deployment
- Experience with CI/CD pipelines and DevOps practices
- Track record of delivering scalable, production-ready applications

I would welcome the opportunity to discuss how my skills align with your team's needs. 
I am available for a call at your earliest convenience.

Best regards,
{{candidate_name}}
{{your_email}}
```

## 2. Follow-up Email (3 days later)

**Subject:** Re: {{job_title}} at {{company_name}} - Following Up

```html
Dear Hiring Manager,

I hope this message finds you well. I wanted to follow up on my previous application for the {{job_title}} position.

I remain very interested in this opportunity and would be delighted to provide any additional information you might need. 
My experience with [specific technology from job posting] would be particularly valuable in addressing [specific challenge from job posting].

Would you be available for a brief conversation this week?

Thank you for considering my application.

Best regards,
{{candidate_name}}
```

## 3. Follow-up Email (1 week later)

**Subject:** Continued Interest in {{job_title}} Role

```html
Dear Hiring Manager,

I wanted to reach out once more regarding the {{job_title}} position at {{company_name}}. 
Your company's focus on [company mission/focus] particularly resonates with me, as it aligns with my professional values.

I am very enthusiastic about contributing to your team and would appreciate any feedback on my candidacy.

I'm available to discuss how I can add value to {{company_name}} at your convenience.

Best regards,
{{candidate_name}}
```

## 4. Referral Outreach Email

**Subject:** Introduction via {{referral_name}} - {{job_title}} Opportunity

```html
Dear {{hiring_manager_name}},

{{referral_name}} recently mentioned that you are looking for a {{job_title}}. 
Given my background in [relevant experience], I thought it would be worth reaching out.

I have [X years] of experience in [relevant field] and have successfully [key accomplishment]. 
I believe my skills and experience would be valuable to {{company_name}}.

Would you have time for a brief conversation next week?

Thank you,
{{candidate_name}}
```

## 5. Thank You Email (Post-Interview)

**Subject:** Thank You - {{job_title}} Interview Discussion

```html
Dear {{interviewer_name}},

Thank you for taking the time to speak with me today about the {{job_title}} position at {{company_name}}. 
I genuinely enjoyed our conversation and learning more about the exciting projects your team is working on.

The opportunity to work on [specific project/challenge discussed] particularly excited me, and I am confident that my experience with 
[relevant skill] would enable me to make immediate contributions.

I am very interested in this opportunity and look forward to hearing from you.

Best regards,
{{candidate_name}}
```

## 6. Counter-Offer Negotiation Email

**Subject:** Discussion of Offer - {{job_title}} Position

```html
Dear {{hiring_manager_name}},

Thank you for the offer for the {{job_title}} position. I am truly excited about the opportunity to join {{company_name}}.

I would like to discuss the compensation package. Based on my research of market rates for this role and my experience level, 
I believe a salary in the range of $[X] - $[Y] would be more aligned with industry standards. 
I am also interested in discussing [other benefits: remote work, signing bonus, etc.].

I remain very enthusiastic about this role and believe we can reach an agreement that works for both parties.

Would you be available to discuss this further?

Best regards,
{{candidate_name}}
```

## 7. Decline Offer Email

**Subject:** {{job_title}} Position - Decision

```html
Dear {{hiring_manager_name}},

Thank you for the opportunity and the offer for the {{job_title}} position at {{company_name}}. 
I have thoroughly considered this opportunity and appreciate the time your team invested in the interview process.

After careful consideration, I have decided to pursue another opportunity that aligns more closely with my career goals at this time.

I have great respect for {{company_name}} and would welcome the opportunity to stay in touch for future positions.

Thank you again for this opportunity.

Best regards,
{{candidate_name}}
```

## 8. Networking Email

**Subject:** Connecting with You - {{your_expertise_area}}

```html
Dear {{contact_name}},

I came across your profile and was impressed by your work in [specific achievement/project]. 
I am also passionate about [shared interest/technology] and would love to connect and learn from your experience.

I am currently exploring opportunities in [industry/role type] and would appreciate any insights you might be willing to share.

Would you be open to a brief 15-minute call next week?

Best regards,
{{candidate_name}}
```

## Best Practices

1. **Personalization**: Always include the hiring manager's name and specific details about the company/role
2. **Keep it Concise**: Aim for 100-150 words in the email body
3. **Professional Tone**: Maintain a balance between formal and personable
4. **Proofread**: Check for spelling and grammar errors
5. **Mobile-Friendly**: Test templates on mobile devices
6. **Timing**: Send emails at optimal times (Tuesday-Thursday, 9-11 AM)
7. **Follow Up**: Wait 3-7 days before following up
8. **Customize Subject Lines**: Increase open rates with relevant, specific subjects

## A/B Testing

Consider testing different email templates to find what works best:
- Subject line variations
- Email length (short vs. detailed)
- Tone variations (casual vs. formal)
- Different call-to-action phrases

## Integration with n8n Workflow

These templates are used in the n8n workflow with the following nodes:
- **Gmail Node**: Sends personalized emails with template variables replaced
- **Variable Replacement**: n8n automatically replaces {{variable}} with actual data
- **Scheduling**: Emails are queued and sent at optimal times based on timezone

## Customization Tips

1. Replace generic greetings with specific names when available
2. Add company-specific details to show you researched the company
3. Highlight how your skills match the job requirements
4. Include relevant links to portfolio, GitHub, or LinkedIn
5. Keep formatting clean and professional
6. Test send to yourself first before automated sending
