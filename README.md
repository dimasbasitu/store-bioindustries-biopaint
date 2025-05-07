# Dokumentasi Developer - Webstore Bioindustries

Dokumentasi ini mencakup keseluruhan sistem dan fungsionalitas dari website e-commerce [store.bioindustries.co.id](https://store.bioindustries.co.id), yang dibangun menggunakan WordPress dan WooCommerce. Seluruh pengembangan dilakukan oleh satu orang developer (Anda sendiri).

## 1. Halaman Utama
- Menampilkan banner promosi, kategori produk, produk unggulan, dan ajakan untuk berkonsultasi.
- Call to action menuju WhatsApp Customer Service.

## 2. Kategori Produk
### a. Wood Finishing & Care
- Produk untuk melindungi, memperindah, dan merawat permukaan kayu.

### b. Adhesive
- Lem kayu dan perekat lainnya untuk keperluan struktural dan kerajinan.

### c. Wood Treatment
- Produk anti-jamur, anti-rayap, dan pengawet kayu.

## 3. Brand yang Ditampilkan
- **Biovarnish** – Cat water based untuk hasil natural finishing.
- **Bioduco** – Cat duco dengan warna solid, cocok untuk proyek kreatif.
- **Biopolish** – Produk perawatan dan poles kayu alami.
- **Biocide** – Pengawet kayu dan anti-jamur.
- **Orchid Enamel Paint** – Cat enamel serbaguna untuk besi dan kayu.
- **Silica Gel** – Produk tambahan untuk pengawetan barang.

## 4. Fitur Utama Website
- Checkout dengan transfer manual (BCA) dan sistem validasi manual.
- Pengiriman terintegrasi dengan **KiriminAja**, mendukung COD dan non-COD.
- Custom tampilan dashboard pelanggan: menampilkan riwayat pesanan manual, salin rekening, dan instruksi pembayaran.
- Kode kupon digital, dapat digunakan langsung dari dashboard.

## 5. Teknologi yang Digunakan
- **Platform CMS**: WordPress
- **Plugin E-Commerce**: WooCommerce
- **Tema**: Orchid Store (dengan modifikasi di `functions.php`)
- **Bahasa Pemrograman**: PHP, HTML, CSS, JavaScript
- **Integrasi Eksternal**: KiriminAja API (pengiriman), WhatsApp API (kontak CS)

## 6. Struktur Direktori dan File Penting
```
wp-content/
├── themes/
│   └── orchid-store/
│       ├── functions.php          # Berisi seluruh fungsi custom
│       └── ...
├── plugins/
│   ├── woocommerce/              # Plugin WooCommerce
│   └── kiriminaja-integrator/    # Jika menggunakan plugin khusus
└── uploads/                      # Tempat media produk dan aset
```

## 7. Fungsi Kustom di functions.php
File `functions.php` berisi modifikasi utama untuk:
- Mengatur akses role pengguna (shop manager)
- Menampilkan informasi tambahan di dashboard pelanggan
- Menangani input manual seperti nomor resi dan riwayat pesanan
- Menambahkan shortcut (dashboard cards) di akun pelanggan
- Menangani redirect setelah checkout ke halaman instruksi pembayaran

### Daftar Fungsi (kecuali fungsi default tema Orchid Store):
- `copyRekening`
- `custom_my_account_dashboard_cards`
- `handle_payment_callback`
- `kiriminaja_access_for_shop_manager`
- `kupon_card_styles`
- `redirect_to_custom_payment_page`
- `tampilkan_berat_di_checkout`
- `tampilkan_instruksi_pembayaran`
- `tampilkan_kupon_card`
- `tampilkan_menu_kupon`
- `tampilkan_riwayat_pesanan_manual`

## Penjelasan Fungsi Kustom
### `copyRekening`
```php
function copyRekening() {
        var rekening = document.getElementById("nomor-rekening").innerText;
        var tempInput = document.createElement("input");
        tempInput.value = rekening;
        document.body.appendChild(tempInput);
        tempInput.select();
        tempInput.setSelectionRange(0, 99999);
        document.execCommand("copy");
        document.body.removeChild(tempInput);
        alert("Nomor rekening berhasil disalin!");
    }
    </script>
    <?php
    return ob_get_clean();
}
```
**Penjelasan:**
Menambahkan fitur salin nomor rekening ke clipboard dari halaman instruksi pembayaran.

### `custom_my_account_dashboard_cards`
```php
function custom_my_account_dashboard_cards() {
    if (!is_user_logged_in()) {
        return '<p>Silakan <a href="https://store.bioindustries.co.id/masuk-akun/">login</a> terlebih dahulu untuk melihat akun Anda.</p>';
    }

    $user_id = get_current_user_id();
    $user_info = get_userdata($user_id);
    $orders = wc_get_orders(array(
        'customer_id' => $user_id,
        'limit' => -1,
        'orderby' => 'date',
        'order' => 'DESC',
    ));

    ob_start();
    ?>
    <style>
        .dashboard-card {
            background: #fff;
            border-radius: 8px;
            padding: 16px;
            box-shadow: 0 2px 8px rgba(0,0,0,0.08);
            margin-bottom: 20px;
        }
        .dashboard-card h3 {
            margin-bottom: 12px;
            font-size: 18px;
        }
        .order-product {
            display: flex;
            gap: 12px;
            margin-bottom: 12px;
            align-items: center;
        }
        .order-product img {
            width: 60px;
            height: auto;
            border-radius: 6px;
        }
        .order-meta {
            font-size: 14px;
            color: #555;
        }
        .button-small {
            padding: 6px 10px;
            font-size: 13px;
            border: none;
            border-radius: 4px;
            background-color: #1f5132;
            color: white;
            margin-right: 6px;
            text-decoration: none;
        }
        .button-small:hover {
            background-color: #163d25;
        }
    </style>

    <div class="dashboard-card">
        <h3>👤 Profil</h3>
        <p><strong>Nama:</strong> <?php echo esc_html($user_info->display_name); ?></p>
        <p><strong>Email:</strong> <?php echo esc_html($user_info->user_email); ?></p>
    </div>

    <div class="dashboard-card">
        <h3>📦 Ringkasan Pesanan</h3>
        <?php if ($orders) : ?>
            <?php foreach ($orders as $order) : ?>
                <div class="order-meta">
                    <p><strong>ID Pesanan:</strong> #<?php echo $order->get_id(); ?> | <strong>Status:</strong> <?php echo wc_get_order_status_name($order->get_status()); ?></p>
                    <p><strong>Metode Pembayaran:</strong> <?php echo $order->get_payment_method_title(); ?></p>
                    <p><strong>Metode Pengiriman:</strong> <?php echo $order->get_shipping_method(); ?></p>
                </div>
                <?php foreach ($order->get_items() as $item_id => $item) :
                    $product = $item->get_product();
                    if (!$product) continue;
                    $thumbnail = $product->get_image(array(300, 300), array('style' => 'width: 100px; height: auto;'));
                    $product_url = get_permalink($product->get_id());
                ?>
                    <div class="order-product">
                        <a href="<?php echo esc_url($product_url); ?>"><?php echo $thumbnail; ?></a>
                        <div>
                            <a href="<?php echo esc_url($product_url); ?>"><?php echo esc_html($item->get_name()); ?></a><br>
                            <span>Qty: <?php echo $item->get_quantity(); ?> | Total: <?php echo wc_price($item->get_total()); ?></span>
                        </div>
                    </div>
                <?php endforeach; ?>
                <hr>
            <?php endforeach; ?>
        <?php else : ?>
            <p>Belum ada pesanan.</p>
        <?php endif; ?>
    </div>

    <div class="dashboard-card">
        <h3>🚪 Logout</h3>
        <a href="<?php echo esc_url(wc_logout_url()); ?>" class="button-small">Keluar dari Akun</a>
    </div>
    <?php
    return ob_get_clean();
}
```
**Penjelasan:**
Menambahkan kartu dashboard kustom pada halaman akun pelanggan WooCommerce.

### `handle_payment_callback`
```php
function handle_payment_callback($request) {
    $body = $request->get_json_params(); // Ambil data JSON dari POST

    $order_id = sanitize_text_field($body['order_id']);
    $status = sanitize_text_field($body['transaction_status']);

    $order = wc_get_order($order_id); // Cari order berdasarkan ID

    if ($order) {
        // Perbarui status pesanan berdasarkan status dari payment gateway
        if ($status === 'settlement') {
            $order->payment_complete();
            $order->add_order_note('Pembayaran sukses via callback.');
        } elseif ($status === 'expire') {
            $order->update_status('failed', 'Pembayaran expired via callback.');
        } else {
            $order->add_order_note('Callback diterima, status: ' . $status);
        }

        return new WP_REST_Response(['message' => 'Callback berhasil diproses.'], 200);
    }

    return new WP_REST_Response(['message' => 'Order tidak ditemukan.'], 404);
}
```
**Penjelasan:**
Menangani callback dari sistem pembayaran manual, memperbarui status pesanan.

### `kiriminaja_access_for_shop_manager`
```php
function kiriminaja_access_for_shop_manager() {
    if ( current_user_can( 'shop_manager' ) ) {
        global $submenu;

        // Cari menu "KiriminAja" di admin dan kasih akses ke Shop Manager
        if ( isset( $submenu['woocommerce'] ) ) {
            foreach ( $submenu['woocommerce'] as $key => $menu_item ) {
                if ( strpos( strtolower( $menu_item[0] ), 'kiriminaja' ) !== false ) {
                    $submenu['woocommerce'][$key][1] = 'manage_woocommerce'; // Shop Manager punya 'manage_woocommerce' capability
                }
            }
        }
    }
}
```
**Penjelasan:**
Memberikan akses menu KiriminAja di dashboard untuk role Shop Manager.

### `kupon_card_styles`
```php
function kupon_card_styles() {
    echo '<style>
        .kupon-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
            gap: 20px;
            margin-top: 20px;
        }
        .kupon-card {
            background: #ffffff;
            border: 1px solid #dcdcdc;
            border-radius: 8px;
            padding: 20px;
            box-shadow: 0 3px 10px rgba(0,0,0,0.05);
            transition: all 0.3s ease;
        }
        .kupon-card:hover {
            border-color: #007cba;
            box-shadow: 0 4px 14px rgba(0,124,186,0.2);
        }
        .kupon-code span {
            font-weight: bold;
            color: #007cba;
        }
        .kupon-desc {
            margin: 10px 0;
            color: #555;
        }
        .kupon-button {
            background-color: #28a745;
            color: #fff;
            padding: 10px 15px;
            display: inline-block;
            margin-top: 10px;
            text-decoration: none;
            border-radius: 4px;
        }
        .kupon-button:hover {
            background-color: #000000;
        }
    </style>';
}
```
**Penjelasan:**
Fungsi tambahan untuk menyesuaikan perilaku WooCommerce sesuai kebutuhan webstore.

### `redirect_to_custom_payment_page`
```php
function redirect_to_custom_payment_page( $order_id ){
    if ( ! $order_id ) return;

    $order = wc_get_order( $order_id );

    if ( $order->get_payment_method() == 'bacs' ) {
        if ( $order->get_status() !== 'pending' ) {
            $order->update_status( 'pending', 'Menunggu pembayaran transfer bank manual.' );
        }

        wp_safe_redirect( home_url( '/instruksi-pembayaran/?order_id=' . $order_id ) );
        exit;
    }
}
```
**Penjelasan:**
Mengalihkan pelanggan ke halaman instruksi pembayaran setelah checkout.

### `tampilkan_berat_di_checkout`
```php
function tampilkan_berat_di_checkout( $product_name, $cart_item, $cart_item_key ) {
    if ( is_checkout() || is_cart() ) {
        $product = $cart_item['data'];
        $weight = $product->get_weight();
        if ( $weight ) {
            $product_name .= sprintf( '<br><small>Berat: %s %s</small>', $weight, get_option( 'woocommerce_weight_unit' ) );
        }
    }
    return $product_name;
}
```
**Penjelasan:**
Menampilkan elemen tambahan seperti kupon, berat, atau riwayat pesanan di halaman customer.

### `tampilkan_instruksi_pembayaran`
```php
function tampilkan_instruksi_pembayaran() {
    if ( ! isset( $_GET['order_id'] ) ) {
        return '<p>Order ID tidak ditemukan.</p>';
    }

    $order_id = intval( $_GET['order_id'] );
    $order = wc_get_order( $order_id );

    if ( ! $order ) {
        return '<p>Order tidak valid.</p>';
    }

    ob_start();
    ?>
    <style>
    .rekening-info {
        text-align: center;
        margin-bottom: 20px;
    }
    .rekening-info img {
        width: auto;
        max-width: 200px;
        max-height: 80px;
        height: auto;
        margin-bottom: 10px;
        display: block;
        margin-left: auto;
        margin-right: auto;
    }
    .rekening-details {
        font-size: 16px;
    }
    .total-transfer {
        text-align: center;
        font-size: 16px;
        margin-bottom: 20px;
    }
    .amount-box {
        display: inline-block;
        background: #f0f0f0;
        padding: 10px 20px;
        border-radius: 8px;
        font-size: 24px;
        font-weight: bold;
        margin-top: 10px;
    }
    .copy-btn {
        display: inline-block;
        margin-top: 5px;
        padding: 5px 10px;
        font-size: 14px;
        cursor: pointer;
        background-color: #0073aa;
        color: #fff;
        border: none;
        border-radius: 4px;
    }
    .copy-btn:hover {
        background-color: #005177;
    }
    </style>


    <div class="rekening-info">
        <img src="https://store.bioindustries.co.id/wp-content/uploads/2025/04/960px-Bank_Central_Asia.svg_.png" alt="Logo BCA">
        <div class="rekening-details">
            <p><strong>Bank:</strong> BCA</p>
            <p><strong>Atas Nama:</strong> CV BIO INDUSTRIES</p>
            <p><strong>Nomor Rekening:</strong> 
                <span id="nomor-rekening">0374080403</span> 
                <button class="copy-btn" onclick="copyRekening()">Salin</button>
            </p>
        </div>
    </div>

    <div class="total-transfer">
        <p><strong>Total yang harus ditransfer:</strong></p>
        <div class="amount-box"><?php echo wc_price( $order->get_total() ); ?></div>
    </div>

    <br>
    <p><strong>Mohon screenshot bukti pembayaran anda, dan upload pada kolom dibawah.</strong></p>

    <h3>Upload Bukti Pembayaran</h3>
    <form action="" method="post" enctype="multipart/form-data">
        <input type="file" name="bukti_transfer" required>
        <input type="hidden" name="order_id" value="<?php echo esc_attr( $order_id ); ?>">
        <br><br>
        <button type="submit" name="submit_bukti" style="padding:8px 16px; background-color:#217a00; color:#fff; border:none; border-radius:4px;">Upload Bukti Transfer</button>
    </form>

    <br>
    <p><strong>Pembayaran anda akan diproses dalam 1x24 jam.</strong></p>

    <script>
    function copyRekening() {
        var rekening = document.getElementById("nomor-rekening").innerText;
        var tempInput = document.createElement("input");
        tempInput.value = rekening;
        document.body.appendChild(tempInput);
        tempInput.select();
        tempInput.setSelectionRange(0, 99999);
        document.execCommand("copy");
        document.body.removeChild(tempInput);
        alert("Nomor rekening berhasil disalin!");
    }
    </script>
    <?php
    return ob_get_clean();
}
```
**Penjelasan:**
Menampilkan elemen tambahan seperti kupon, berat, atau riwayat pesanan di halaman customer.

### `tampilkan_kupon_card`
```php
function tampilkan_kupon_card() {
    $coupons = get_posts(array(
        'post_type' => 'shop_coupon',
        'posts_per_page' => -1,
        'post_status' => 'publish'
    ));

    if (empty($coupons)) return '<p>Tidak ada kupon aktif saat ini.</p>';

    $output = '<div class="kupon-grid">';
    foreach ($coupons as $coupon_post) {
        $coupon = new WC_Coupon($coupon_post->post_title);

        // Skip jika expired
        if ($coupon->get_date_expires() && $coupon->get_date_expires()->getTimestamp() < time()) continue;

        $code = esc_html($coupon->get_code());
        $amount = wc_price($coupon->get_amount());
        $desc = esc_html($coupon->get_description());

        $output .= "
        <div class='kupon-card'>
            <div class='kupon-body'>
                <h3 class='kupon-code'>Kode: <span>{$code}</span></h3>
                <p class='kupon-desc'>{$desc}</p>
                <p class='kupon-amount'>Potongan: <strong>{$amount}</strong></p>
                <a href='" . wc_get_cart_url() . "?coupon={$code}' class='kupon-button'>Klaim Sekarang</a>
            </div>
        </div>";
    }
    $output .= '</div>';

    return $output;
}
```
**Penjelasan:**
Menampilkan elemen tambahan seperti kupon, berat, atau riwayat pesanan di halaman customer.

### `tampilkan_menu_kupon`
```php
function tampilkan_menu_kupon() {
    add_submenu_page('woocommerce', 'Coupons', 'Coupons', 'manage_woocommerce', 'edit.php?post_type=shop_coupon');
}
```
**Penjelasan:**
Menampilkan elemen tambahan seperti kupon, berat, atau riwayat pesanan di halaman customer.

### `tampilkan_riwayat_pesanan_manual`
```php
function tampilkan_riwayat_pesanan_manual() {
    if ( ! is_user_logged_in() ) {
        return '<p>Silakan <a href="https://store.bioindustries.co.id/masuk-akun/">login</a> untuk melihat riwayat pesanan Anda.</p>';
    }

    if ( isset($_POST['cancel_order']) && isset($_POST['cancel_order_id']) ) {
        $order_id = intval($_POST['cancel_order_id']);
        $order = wc_get_order( $order_id );

        if ( $order && $order->get_customer_id() == get_current_user_id() ) {
            $order->update_status( 'cancelled', 'Pesanan dibatalkan oleh pelanggan.' );
            wp_safe_redirect( esc_url( $_SERVER['REQUEST_URI'] ) );
            exit;
        }
    }

    if ( isset($_POST['order_completed']) && isset($_POST['order_id']) ) {
        $order_id = intval($_POST['order_id']);
        $order = wc_get_order( $order_id );

        if ( $order && $order->get_customer_id() == get_current_user_id() ) {
            $order->update_status( 'completed', 'Pesanan ditandai selesai oleh pelanggan.' );
            wp_safe_redirect( esc_url( $_SERVER['REQUEST_URI'] ) );
            exit;
        }
    }

    $user_id = get_current_user_id();
    $customer_orders = wc_get_orders( array(
        'customer_id' => $user_id,
        'limit'       => -1,
        'orderby'     => 'date',
        'order'       => 'DESC',
        'status'      => array( 'processing', 'completed', 'on-hold', 'pending', 'cancelled', 'refunded' )
    ) );

    if ( empty( $customer_orders ) ) {
        return '<p>Belum ada pesanan yang tercatat.</p>';
    }

    $output = '<div style="display: flex; flex-direction: column; gap: 20px;">';

    foreach ( $customer_orders as $order ) {
        $access_key = '';
        $items_output = '<div style="margin: 10px 0;">';

        foreach ( $order->get_items() as $item ) {
            $product = $item->get_product();
            $product_name = $item->get_name();
            $quantity = $item->get_quantity();
            $subtotal = wc_price( $item->get_total() );
            $product_url = $product ? get_permalink( $product->get_id() ) : '#';
            $thumbnail = $product ? '<a href="' . esc_url( $product_url ) . '">' . $product->get_image( array( 300, 300 ), [ 'style' => 'width: 200px; height: auto; border-radius: 6px;' ] ) . '</a>' : '';

            $items_output .= '<div style="display: flex; align-items: center; gap: 12px; margin-bottom: 12px;">
                <div>' . $thumbnail . '</div>
                <div><a href="' . esc_url( $product_url ) . '" style="text-decoration: none; color: #111;">' . $product_name . '</a> × ' . $quantity . ' = ' . $subtotal . '</div>
            </div>';
        }
        $items_output .= '</div>';

        $order_date = $order->get_date_created();
        $formatted_date = $order_date ? $order_date->date_i18n( 'j F Y, H:i' ) : '';

        $output .= '<div style="border: 1px solid #ddd; border-radius: 8px; padding: 16px; box-shadow: 0 1px 3px rgba(0,0,0,0.1);">';
        $output .= '<strong>Pesanan #' . $order->get_id() . '</strong> – ' . $formatted_date;
        $output .= $items_output;
        $output .= '<p>Status: <strong>' . wc_get_order_status_name( $order->get_status() ) . '</strong><br>';
        $output .= 'Metode Pembayaran: ' . $order->get_payment_method_title() . '<br>';
        $output .= 'Metode Pengiriman: ' . $order->get_shipping_method() . '<br>';

        $discount_total = is_callable( [ $order, 'get_total_discount' ] ) ? $order->get_total_discount() : 0;
        if ( is_numeric($discount_total) && floatval($discount_total) > 0 ) {
            $output .= 'Diskon Kupon: -' . wc_price( $discount_total ) . '<br>';
            $coupons = $order->get_coupon_codes();
            if ( ! empty( $coupons ) ) {
                $output .= 'Kode Kupon: ' . implode( ', ', $coupons ) . '<br>';
            }
        }

        $output .= 'Total: <strong>' . $order->get_formatted_order_total() . '</strong></p>';

        // Link download invoice PDF
        if ( class_exists( 'WPO_WCPDF_Invoice' ) && method_exists( 'WPO_WCPDF_Invoice', 'get_instance' ) ) {
            $invoice = WPO_WCPDF_Invoice::get_instance();
            if ( method_exists( $invoice, 'get_document' ) ) {
                $document = $invoice->get_document( $order );
                if ( $document && method_exists( $document, 'get_access_key' ) ) {
                    $access_key = $document->get_access_key();
                    update_post_meta( $order->get_id(), '_wcpdf_invoice_access_key', $access_key );
                }
            }
        }

        if ( isset( $access_key ) && ! empty( $access_key ) ) {
            $pdf_url = add_query_arg( array(
                'action'        => 'generate_wpo_wcpdf',
                'document_type' => 'invoice',
                'order_ids'     => $order->get_id(),
                'access_key'    => $access_key
            ), admin_url( 'admin-ajax.php' ) );

            $output .= '<a href="' . esc_url( $pdf_url ) . '" target="_blank" style="background: #2563eb; color: white; padding: 8px 16px; border-radius: 4px; text-decoration: none; display:inline-block; margin-top: 10px; margin-right:10px;">Download Invoice PDF</a>';
        }

        // Tombol "Pesanan Diterima"
        if ( $order->get_status() === 'processing' ) {
            $output .= '<form method="post" style="margin-top: 10px; display:inline-block; margin-right:10px;">
                <input type="hidden" name="order_id" value="' . $order->get_id() . '" />
                <input type="submit" name="order_completed" value="Pesanan Diterima" style="background: #14532d; color: white; padding: 8px 16px; border: none; border-radius: 4px; cursor: pointer;" onclick="return confirm(\'Yakin pesanan sudah diterima?\');" />
            </form>';
        }

        // Tombol "Batalkan Pesanan"
        if ( in_array( $order->get_status(), array( 'pending', 'processing' ) ) ) {
            $output .= '<form method="post" style="margin-top: 10px; display:inline-block; margin-right:10px;">
                <input type="hidden" name="cancel_order_id" value="' . $order->get_id() . '" />
                <input type="submit" name="cancel_order" value="Batalkan Pesanan" style="background: #991b1b; color: white; padding: 8px 16px; border: none; border-radius: 4px; cursor: pointer;" onclick="return confirm(\'Yakin ingin membatalkan pesanan ini?\');" />
            </form>';
        }

        // Tombol "Lanjutkan Pembayaran" untuk Transfer Bank Pending
        if ( $order->get_payment_method() === 'bacs' && $order->get_status() === 'pending' ) {
            $output .= '<a href="' . esc_url( home_url( '/instruksi-pembayaran/?order_id=' . $order->get_id() ) ) . '" style="background: #f59e0b; color: white; padding: 8px 16px; border-radius: 4px; text-decoration: none; display:inline-block; margin-top: 10px; margin-right:10px;">Lanjutkan Pembayaran</a>';
        }

        $output .= '</div>';
    }

    $output .= '</div>';
    return $output;
}
```
**Penjelasan:**
Menampilkan elemen tambahan seperti kupon, berat, atau riwayat pesanan di halaman customer.

## 8. Integrasi Pihak Ketiga

### a. KiriminAja (Pengiriman)
- KiriminAja digunakan sebagai agregator pengiriman (termasuk COD dan non-COD).
- Sistem ini mengatur data pengiriman langsung dari dashboard admin WooCommerce.
- Akses diberikan ke peran **Shop Manager** menggunakan fungsi `kiriminaja_access_for_shop_manager`.

### b. Metode Pembayaran Manual (Transfer BCA)
- Sistem pembayaran tidak menggunakan payment gateway otomatis.
- Setelah checkout, pelanggan diarahkan ke halaman instruksi pembayaran manual.
- Dilengkapi fitur salin rekening (`copyRekening`) dan status pesanan diubah manual oleh admin.

---
Dokumentasi ini dapat diperluas jika ada perubahan sistem, penambahan metode pembayaran baru, atau integrasi eksternal tambahan.