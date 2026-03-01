# Troubleshooting Upload & View PDF Issues

## Common Issues and Solutions

### 1. Upload Error: "Access Denied" or "Permission Denied"

**Solution:** Check Supabase Storage Policies

Run this SQL in Supabase SQL Editor:

```sql
-- Check if bucket exists
SELECT * FROM storage.buckets WHERE id = 'office-forms';

-- If bucket doesn't exist, create it
INSERT INTO storage.buckets (id, name, public)
VALUES ('office-forms', 'office-forms', false)
ON CONFLICT (id) DO NOTHING;

-- Drop old policies if they exist
DROP POLICY IF EXISTS "Only admins can upload files" ON storage.objects;
DROP POLICY IF EXISTS "Only admins can download files" ON storage.objects;
DROP POLICY IF EXISTS "Only admins can delete files" ON storage.objects;

-- Create new storage policies
CREATE POLICY "Admins can upload files"
  ON storage.objects FOR INSERT
  WITH CHECK (
    bucket_id = 'office-forms' AND 
    EXISTS (SELECT 1 FROM user_roles WHERE user_id = auth.uid() AND role = 'ADMIN')
  );

CREATE POLICY "Admins can view files"
  ON storage.objects FOR SELECT
  USING (
    bucket_id = 'office-forms' AND 
    EXISTS (SELECT 1 FROM user_roles WHERE user_id = auth.uid() AND role = 'ADMIN')
  );

CREATE POLICY "Admins can update files"
  ON storage.objects FOR UPDATE
  USING (
    bucket_id = 'office-forms' AND 
    EXISTS (SELECT 1 FROM user_roles WHERE user_id = auth.uid() AND role = 'ADMIN')
  );

CREATE POLICY "Admins can delete files"
  ON storage.objects FOR DELETE
  USING (
    bucket_id = 'office-forms' AND 
    EXISTS (SELECT 1 FROM user_roles WHERE user_id = auth.uid() AND role = 'ADMIN')
  );
```

### 2. View/Download Error: "File not found"

**Check:**
- File was uploaded successfully (check Supabase Storage dashboard)
- File URL is stored correctly in database
- Storage bucket name is correct ('office-forms')

### 3. PDF Not Opening in Browser

**Solution:** The PDF should open in a new tab. If it doesn't:
- Check browser popup blocker settings
- Try downloading the file instead
- Check browser console for errors (F12)

### 4. File Upload Stuck on "Uploa