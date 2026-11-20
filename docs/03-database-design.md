# طراحی اولیه پایگاه داده

این سند طرح مفهومی نسخه اول است. نام فیلدها و جزئیات نهایی پس از تثبیت نیازمندی‌ها و تولید Migrationها قطعی می‌شوند.

## اصول طراحی
- کلید اصلی عددی یا UUID بر اساس تصمیم نهایی؛ در نسخه اول شناسه عددی مناسب است.
- جداول عملیاتی با InnoDB و charset برابر `utf8mb4`.
- زمان‌ها در پایگاه داده با سیاست زمانی یکپارچه ذخیره و در رابط کاربری محلی‌سازی شوند.
- حذف منطقی برای داده‌هایی که سابقه و ممیزی آن‌ها لازم است.
- شاخص‌گذاری روی کلیدهای خارجی، وضعیت‌ها و ستون‌های جست‌وجوی پرتکرار.
- اطلاعات کودک حداقلی و فقط به میزان نیاز محصول ذخیره شود.

## کاربران و کودکان
### users
حساب والد/مدیر: `id`, `name`, `email`, `mobile`, `password_hash`, `role`, `status`, `created_at`, `updated_at`.

### children
پروفایل کودک: `id`, `parent_id`, `display_name`, `birth_date` یا بازه سنی، `age_group`, `status`, `created_at`, `updated_at`.

رابطه: هر کاربر والد می‌تواند چند پروفایل کودک داشته باشد. کنترل مالکیت والد در تمام درخواست‌های مرتبط با کودک الزامی است.

## مهارت‌ها
### skill_categories
`id`, `title`, `slug`, `description`, `sort_order`, `status`.

### skills
`id`, `category_id`, `title`, `slug`, `description`, `age_group`, `status`.

رابطه: هر دسته چند مهارت و هر مهارت متعلق به یک دسته است.

## بسته‌ها و محتوای آموزشی
### content_packages
`id`, `title`, `slug`, `topic`, `age_group`, `category_id`, `status`, `scheduled_at`, `published_at`, `created_at`.

### contents
`id`, `package_id`, `content_type`, `title`, `body`, `status`, `version`, `created_at`, `updated_at`.

انواع محتوا می‌تواند `lesson`, `quiz`, `mission` باشد. در صورت نیاز به ساختار دقیق‌تر، آزمون و مأموریت در جداول مستقل نگهداری می‌شوند.

### content_versions
`id`, `content_id`, `version_number`, `payload_json`, `created_by`, `created_at`, `change_note`.

نسخه‌های قبلی برای ممیزی و بازگشت نگهداری شوند.

## آزمون
### quizzes
`id`, `package_id`, `title`, `instructions`, `status`.

### quiz_questions
`id`, `quiz_id`, `question_text`, `question_type`, `sort_order`, `explanation`.

### quiz_options
`id`, `question_id`, `option_text`, `is_correct`, `sort_order`.

پاسخ صحیح نباید در endpoint عمومی کودک افشا شود. بررسی پاسخ باید سمت سرور انجام شود.

### quiz_attempts
`id`, `quiz_id`, `child_id`, `started_at`, `submitted_at`, `score`, `status`.

### quiz_answers
`id`, `attempt_id`, `question_id`, `selected_option_id`, `answer_text`, `is_correct`.

## مأموریت و پیشرفت
### missions
`id`, `package_id`, `title`, `instructions`, `completion_criteria`, `status`.

### mission_submissions
`id`, `mission_id`, `child_id`, `response_text`, `attachment_id`, `status`, `submitted_at`, `reviewed_at`.

### child_progress
`id`, `child_id`, `skill_id`, `practice_count`, `last_practiced_at`, `progress_note`, `updated_at`.

در MVP نمایش پیشرفت بر اساس فعالیت‌های انجام‌شده و مهارت‌های تمرین‌شده است؛ از امتیازدهی پیچیده یا ادعای سنجش علمی توانایی پرهیز می‌شود.

## رسانه
### media_assets
`id`, `package_id`, `media_type`, `storage_path`, `mime_type`, `file_size`, `checksum`, `status`, `created_at`.

فایل‌های تولیدشده تا پیش از تأیید در فضای خصوصی نگهداری شوند.

## تأیید و انتشار
### content_approvals
`id`, `package_id`, `reviewer_id`, `decision`, `comment`, `created_at`.

### content_schedules
`id`, `package_id`, `publish_at`, `status`, `created_at`.

تأیید بسته باید تنها زمانی ممکن باشد که اجزای اجباری کامل و اعتبارسنجی شده باشند.

## صف پردازش
### jobs
`id`, `job_type`, `payload_json`, `status`, `attempts`, `available_at`, `reserved_at`, `finished_at`, `last_error`, `created_at`.

### job_logs
`id`, `job_id`, `level`, `message`, `context_json`, `created_at`.

برای جلوگیری از اجرای هم‌زمان یک job، قفل‌گذاری و انقضای reservation لازم است.

## روابط کلیدی
- `children.parent_id → users.id`
- `skills.category_id → skill_categories.id`
- `content_packages.category_id → skill_categories.id`
- `contents.package_id → content_packages.id`
- `quizzes.package_id → content_packages.id`
- `quiz_questions.quiz_id → quizzes.id`
- `quiz_options.question_id → quiz_questions.id`
- `missions.package_id → content_packages.id`
- `mission_submissions.mission_id → missions.id`
- `mission_submissions.child_id → children.id`
- `child_progress.child_id → children.id`
- `child_progress.skill_id → skills.id`
- `media_assets.package_id → content_packages.id`
- `jobs.id → job_logs.job_id`

## مواردی که پیش از Migration نهایی باید تعیین شوند
- احراز هویت والد با موبایل، ایمیل یا هر دو.
- سیاست نگهداری و حذف اطلاعات کودک.
- پشتیبانی از چند والد برای یک کودک یا فقط یک مالک.
- نوع پاسخ مأموریت و امکان بارگذاری تصویر توسط کودک.
- جزئیات گزارش پیشرفت و سیاست نگهداری تلاش‌های آزمون.
