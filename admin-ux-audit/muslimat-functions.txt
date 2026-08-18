<?php
/**
 * Aalaam Admin Panel — UX Improvements
 * =====================================
 * أضف هذا الكود في functions.php الخاص بالثيم أو أنشئ إضافة مخصصة.
 *
 * كل تغيير مرفق بسببه — انظر تقرير التدقيق (muslimat-ux-audit.md) للتفاصيل.
 */

// ──────────────────────────────────────────────
// ١. توجيه تسجيل الدخول إلى صفحة الدروس
// السبب: الصفحة الرئيسية فارغة — المستخدم يحتاج سياقاً فورياً
// ملاحظة: حل مؤقت حتى تُنفّذ بطاقات الصفحة الرئيسية (رقم ٩)
// ──────────────────────────────────────────────
add_filter('login_redirect', function ($redirect_to, $request, $user) {
    if (isset($user->roles) && is_array($user->roles)) {
        return admin_url('edit.php?post_type=lesson');
    }
    return $redirect_to;
}, 10, 3);


// ──────────────────────────────────────────────
// ٢. إخفاء عناصر القائمة غير الضرورية
// ──────────────────────────────────────────────
add_action('admin_menu', function () {
    // إخفاء "أدوات" — الصفحة فارغة لهذا الدور
    remove_menu_page('tools.php');

    // إخفاء Taxonomy Order من جميع أنواع المحتوى
    // السبب: صفحة إنجليزية بالكامل مع إعلانات ترويجية تكسر ثقة المستخدم
    $post_types = [
        'lesson',
        'scientific-series',
        'article',
        'fatwa',
        'podcast',
        'quote',
        'scientific-lesson',
    ];
    foreach ($post_types as $pt) {
        remove_submenu_page(
            "edit.php?post_type={$pt}",
            "edit.php?post_type={$pt}&page=to-interface-{$pt}"
        );
    }

    // إخفاء صفحات WPML المحجوبة بالصلاحيات
    // السبب: تُظهر "ليس لديك إذن" — لا تعرض ما لا يمكن استخدامه
    remove_submenu_page('tm/menu/main.php', 'wpml-string-translation/menu/string-translation.php');
    remove_submenu_page('tm/menu/main.php', 'tm/menu/settings');
}, 999);


// ──────────────────────────────────────────────
// ٣. تغيير اسم WPML إلى "الترجمة"
// السبب: المستخدم يبحث عن وظيفة لا عن اسم منتج تقني
// ──────────────────────────────────────────────
add_action('admin_menu', function () {
    global $menu;
    if (is_array($menu)) {
        foreach ($menu as &$item) {
            if (isset($item[0]) && strpos($item[0], 'WPML') !== false) {
                $item[0] = 'الترجمة';
            }
        }
    }
}, 1000);


// ──────────────────────────────────────────────
// ٤. إخفاء meta boxes غير ضرورية من صفحات التحرير
// ملاحظة: Rank Math SEO يُطوى (collapsed) بدلاً من إخفائه — انظر رقم ٨
// ──────────────────────────────────────────────
add_action('add_meta_boxes', function () {
    $post_types = get_post_types(['public' => true]);
    foreach ($post_types as $pt) {
        // حقل المؤلف — يعرض اسم حساب النظام مع زر يسمح بتغيير المؤلف بالخطأ
        remove_meta_box('authordiv', $pt, 'normal');

        // حقل الـ slug — تعديله بالخطأ يكسر الروابط، WP يُنشئه تلقائياً
        remove_meta_box('slugdiv', $pt, 'normal');
    }
}, 99);


// ──────────────────────────────────────────────
// ٥. إخفاء أعمدة غير ضرورية من جداول المحتوى
// السبب: تقليل كثافة المعلومات — الإبقاء على الأساسيات
// ──────────────────────────────────────────────
add_action('admin_init', function () {
    $post_types = [
        'lesson',
        'scientific-series',
        'article',
        'fatwa',
        'static-page',
        'podcast',
        'quote',
        'scientific-lesson',
    ];

    foreach ($post_types as $pt) {
        add_filter("manage_{$pt}_posts_columns", function ($columns) {
            // SEO columns — يعرض "N/A" في الجدول، SEO يُدار من صفحة التحرير
            unset($columns['rank_math_seo_details']);
            unset($columns['wpseo-score']);
            unset($columns['wpseo-title']);

            // Content excerpt — المستخدم يفتح المنشور لقراءة المحتوى
            unset($columns['content']);

            // WPML language flags — إدارة الترجمة لها قسمها الخاص
            foreach (array_keys($columns) as $key) {
                if (strpos($key, 'icl_translations') !== false) {
                    unset($columns[$key]);
                }
            }

            return $columns;
        });
    }
});


// ──────────────────────────────────────────────
// ٦. إخفاء مُنتقي ألوان الإدارة من الملف الشخصي
// السبب: تغييرها يكسر اتساق العلامة التجارية عبر جميع المستخدمين
// ──────────────────────────────────────────────
remove_action('admin_color_scheme_picker', 'admin_color_scheme_picker');


// ──────────────────────────────────────────────
// ٧. اختصار اسم الدور
// السبب: "مدير لوحة تحكم نظام إدارة المحتوى" = ٧ كلمات — طويل جداً
// ──────────────────────────────────────────────
add_action('init', function () {
    global $wp_roles;
    if (isset($wp_roles->roles)) {
        foreach ($wp_roles->roles as $slug => &$role) {
            if ($role['name'] === 'مدير لوحة تحكم نظام إدارة المحتوى') {
                $wp_roles->roles[$slug]['name'] = 'مدير المحتوى';
                $wp_roles->role_names[$slug] = 'مدير المحتوى';
            }
        }
    }
});


// ──────────────────────────────────────────────
// ٨. طي Rank Math SEO افتراضياً (بدلاً من إخفائه)
// السبب: SEO مهم — بعض المستخدمين يحتاجونه. لكن عرضه مفتوحاً
// بنتيجة "9/100" بلون أحمر يُخيف. الحل: اطوِه افتراضياً.
// من يحتاجه ينقر لفتحه، ومن لا يحتاجه لا يراه.
// ──────────────────────────────────────────────
add_action('admin_footer', function () {
    $screen = get_current_screen();
    if ($screen && $screen->base === 'post') {
        ?>
        <script>
        document.addEventListener('DOMContentLoaded', function() {
            var seoBoxes = document.querySelectorAll('#rank_math_metabox, #rank-math-metabox');
            seoBoxes.forEach(function(box) {
                if (!box.classList.contains('closed')) {
                    box.classList.add('closed');
                }
            });
        });
        </script>
        <?php
    }
});


// ──────────────────────────────────────────────
// ٩. بطاقات روابط سريعة للصفحة الرئيسية
// السبب: الصفحة الرئيسية فارغة — بطاقات تعطي المستخدم نقطة بداية فورية
// مثل الشاشة الرئيسية لتطبيق هاتف (Progressive Disclosure)
// ──────────────────────────────────────────────
add_action('wp_dashboard_setup', function () {
    // إزالة الـ widgets الافتراضية
    remove_meta_box('dashboard_primary', 'dashboard', 'side');
    remove_meta_box('dashboard_quick_press', 'dashboard', 'side');
    remove_meta_box('dashboard_right_now', 'dashboard', 'normal');
    remove_meta_box('dashboard_activity', 'dashboard', 'normal');

    // إضافة widget بطاقات الروابط السريعة
    wp_add_dashboard_widget(
        'aalaam_quick_links',
        'الوصول السريع',
        'aalaam_render_quick_links'
    );
});

function aalaam_render_quick_links() {
    $links = [
        ['url' => 'edit.php?post_type=lesson',             'icon' => '📖', 'label' => 'الدروس'],
        ['url' => 'edit.php?post_type=scientific-series',   'icon' => '🎬', 'label' => 'السلاسل العلمية'],
        ['url' => 'edit.php?post_type=article',             'icon' => '📌', 'label' => 'المقالات'],
        ['url' => 'edit.php?post_type=fatwa',               'icon' => '❓', 'label' => 'أسئلة وأجوبة'],
        ['url' => 'edit.php?post_type=static-page',         'icon' => '📄', 'label' => 'الصفحات الثابتة'],
        ['url' => 'edit.php?post_type=podcast',             'icon' => '🎙️', 'label' => 'بودكاست'],
        ['url' => 'edit.php?post_type=quote',               'icon' => '💬', 'label' => 'فوائد'],
        ['url' => 'edit.php?post_type=scientific-lesson',   'icon' => '🎓', 'label' => 'الدروس العلمية'],
        ['url' => 'upload.php',                              'icon' => '🖼️', 'label' => 'الوسائط'],
        ['url' => 'users.php',                               'icon' => '👤', 'label' => 'الأعضاء'],
    ];

    echo '<div class="aalaam-dash-grid">';
    foreach ($links as $link) {
        $url = admin_url($link['url']);
        echo '<a href="' . esc_url($url) . '" class="aalaam-dash-card">';
        echo '<div class="aalaam-dash-icon">' . $link['icon'] . '</div>';
        echo '<span>' . esc_html($link['label']) . '</span>';
        echo '</a>';
    }
    echo '</div>';
}


// ──────────────────────────────────────────────
// ١٠. تحميل ملف CSS المخصص
// ──────────────────────────────────────────────
add_action('admin_enqueue_scripts', function () {
    wp_enqueue_style(
        'muslimat-admin-override',
        get_stylesheet_directory_uri() . '/muslimat-admin-override.css',
        [],
        '1.1.0'
    );
});
