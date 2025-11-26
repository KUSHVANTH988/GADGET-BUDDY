<?php
// index.php - public products page (reads from DB) with titles, prices, captions and modal preview
require_once __DIR__ . '/db.php';
$stmt = $mysqli->prepare('SELECT id, image_path, is_url, redirect_url, title, price, caption FROM products ORDER BY created_at DESC');
$stmt->execute();
$res = $stmt->get_result();
$items = $res->fetch_all(MYSQLI_ASSOC);
?><!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8"/>
  <meta name="viewport" content="width=device-width,initial-scale=1"/>
  <title>Products</title>
  <style>
    :root{
      --accent: #0a6b58;
      --bg-start: #f7fbf9;
      --muted: #6b7175;
      --card-radius: 12px;
      --max-width: 1200px;
    }
    *{box-sizing:border-box}
    html,body{height:100%;margin:0;font-family:Inter,system-ui,Segoe UI,Roboto,Arial;color:#0b2b2a}
    body{background:linear-gradient(180deg,var(--bg-start),#ffffff);padding:44px 18px;display:flex;justify-content:center}
    .wrap{width:100%;max-width:var(--max-width)}
    header{display:flex;align-items:center;justify-content:space-between;margin-bottom:20px}
    header h1{margin:0;color:var(--accent);font-size:28px}
    header p{margin:0;color:var(--muted)}

    .grid{display:grid;grid-template-columns:repeat(1,1fr);gap:22px}
    @media(min-width:640px){ .grid{grid-template-columns:repeat(2,1fr)} }
    @media(min-width:980px){ .grid{grid-template-columns:repeat(3,1fr)} }

    .card{background:#fff;border-radius:var(--card-radius);overflow:hidden;box-shadow:0 8px 20px rgba(11,43,42,0.06);cursor:pointer;transition:transform .25s,box-shadow .25s;display:flex;flex-direction:column}
    .card:hover{transform:translateY(-6px);box-shadow:0 20px 40px rgba(11,43,42,0.08)}
    .thumb{width:100%;height:200px;background:#0b0b0b;display:flex;align-items:center;justify-content:center;overflow:hidden}
    .thumb img{width:100%;height:100%;object-fit:cover;display:block;transition:transform .45s}
    .card:hover .thumb img{transform:scale(1.05)}
    .meta{padding:12px 14px}
    .title{font-weight:700;margin:0 0 6px;font-size:16px;color:#072a26}
    .price{font-weight:800;color:var(--accent);margin-bottom:8px}
    .caption{font-size:13px;color:var(--muted);margin:0}

    /* modal / lightbox */
    .modal-backdrop{position:fixed;inset:0;background:rgba(5,10,8,0.6);display:none;align-items:center;justify-content:center;z-index:9999;padding:20px}
    .modal{background:#fff;border-radius:12px;max-width:900px;width:100%;max-height:90vh;overflow:auto;box-shadow:0 30px 80px rgba(2,8,6,0.5);display:flex;gap:18px;padding:18px}
    .modal-left{flex:1;min-width:320px}
    .modal-left img{width:100%;height:100%;max-height:70vh;object-fit:contain;border-radius:8px;background:#000}
    .modal-right{flex:1;display:flex;flex-direction:column;gap:12px}
    .modal-title{font-size:20px;font-weight:800;color:#072a26}
    .modal-price{font-size:18px;color:var(--accent);font-weight:800}
    .modal-caption{color:var(--muted);line-height:1.4}
    .modal-actions{margin-top:auto;display:flex;gap:12px}
    .btn-visit{background:var(--accent);color:#fff;border:0;padding:10px 14px;border-radius:8px;font-weight:700;text-decoration:none}
    .btn-close{background:#eee;border:0;padding:10px 14px;border-radius:8px;cursor:pointer}

    .empty{padding:28px;border-radius:12px;background:#fff;text-align:center;color:var(--muted);box-shadow:0 8px 20px rgba(11,43,42,0.04)}
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <div>
        <h1>Products</h1>
        <p>Click any card to preview details. Visit the product page from the modal.</p>
      </div>
      <div style="align-self:center;color:var(--muted)"><?php echo count($items); ?> <?php echo (count($items)===1?'product':'products'); ?></div>
    </header>

    <main>
      <?php if (empty($items)): ?>
        <div class="empty">No products yet.</div>
      <?php else: ?>
        <div id="grid" class="grid">
          <?php foreach($items as $it):
            $href = $it['redirect_url'] ? htmlspecialchars($it['redirect_url']) : '#';
            if ((int)$it['is_url'] === 1) {
                $img_src = htmlspecialchars($it['image_path']);
            } else {
                $img_src = UPLOAD_WEBPATH . '/' . htmlspecialchars(basename($it['image_path']));
            }
            $title = htmlspecialchars($it['title']);
            $price = $it['price'] !== null ? number_format((float)$it['price'], 2) : '';
            $caption = htmlspecialchars($it['caption']);
          ?>
            <article class="card" tabindex="0" data-img="<?php echo $img_src; ?>" data-title="<?php echo $title; ?>" data-price="<?php echo $price; ?>" data-caption="<?php echo $caption; ?>" data-href="<?php echo $href; ?>">
              <div class="thumb">
                <img loading="lazy" src="<?php echo $img_src; ?>" alt="<?php echo $title ?: 'Product image'; ?>" onerror="this.src='https://via.placeholder.com/800x600?text=No+Image'"/>
              </div>
              <div class="meta">
                <p class="title"><?php echo $title ?: 'Untitled product'; ?></p>
                <?php if ($price !== ''): ?><div class="price">₹ <?php echo $price; ?></div><?php endif; ?>
                <?php if ($caption !== ''): ?><p class="caption"><?php echo $caption; ?></p><?php endif; ?>
              </div>
            </article>
          <?php endforeach; ?>
        </div>
      <?php endif; ?>
    </main>
  </div>

  <!-- Modal -->
  <div id="modalBackdrop" class="modal-backdrop" role="dialog" aria-hidden="true">
    <div class="modal" role="document" aria-label="Product preview">
      <div class="modal-left">
        <img id="modalImg" src="" alt="">
      </div>
      <div class="modal-right">
        <div id="modalTitle" class="modal-title"></div>
        <div id="modalPrice" class="modal-price"></div>
        <div id="modalCaption" class="modal-caption"></div>
        <div class="modal-actions">
          <a id="modalVisit" class="btn-visit" href="#" target="_blank" rel="noopener noreferrer">Visit</a>
          <button id="modalClose" class="btn-close">Close</button>
        </div>
      </div>
    </div>
  </div>

  <script>
    // open modal when card clicked (or Enter pressed)
    const modalBackdrop = document.getElementById('modalBackdrop');
    const modalImg = document.getElementById('modalImg');
    const modalTitle = document.getElementById('modalTitle');
    const modalPrice = document.getElementById('modalPrice');
    const modalCaption = document.getElementById('modalCaption');
    const modalVisit = document.getElementById('modalVisit');
    const modalClose = document.getElementById('modalClose');

    function openModal(data) {
      modalImg.src = data.img;
      modalImg.alt = data.title || 'Product image';
      modalTitle.textContent = data.title || 'Untitled product';
      modalPrice.textContent = data.price ? '₹ ' + data.price : '';
      modalCaption.textContent = data.caption || '';
      modalVisit.href = data.href || '#';
      modalBackdrop.style.display = 'flex';
      modalBackdrop.setAttribute('aria-hidden','false');
      modalClose.focus();
    }
    function closeModal() {
      modalBackdrop.style.display = 'none';
      modalBackdrop.setAttribute('aria-hidden','true');
      modalImg.src = '';
    }

    document.querySelectorAll('.card').forEach(card => {
      card.addEventListener('click', (e) => {
        const data = {
          img: card.getAttribute('data-img'),
          title: card.getAttribute('data-title'),
          price: card.getAttribute('data-price'),
          caption: card.getAttribute('data-caption'),
          href: card.getAttribute('data-href'),
        };
        openModal(data);
      });
      // keyboard: Enter opens modal
      card.addEventListener('keydown', (e) => { if (e.key === 'Enter') card.click(); });
    });

    modalClose.addEventListener('click', closeModal);
    modalBackdrop.addEventListener('click', (e) => { if (e.target === modalBackdrop) closeModal(); });
    document.addEventListener('keydown', (e) => { if (e.key === 'Escape') closeModal(); });
  </script>
</body>
</html>
