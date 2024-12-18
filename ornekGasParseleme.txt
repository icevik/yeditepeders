function fetchXMLAndSaveProductsToSingleFile() {
  // 1. XML dosyasının URL'si tanımlanıyor.
  var url = 'https://www.perspective.com.tr/feed/google';

  // 2. URL'den veriler alınıyor.
  var response = UrlFetchApp.fetch(url);

  // 3. XML verileri parse ediliyor (çözümleniyor).
  var content = XmlService.parse(response.getContentText());

  // 4. XML'in kök elementi alınıyor.
  var root = content.getRootElement();

  // 5. XML içinde kullanılan namespace (ad alanı) tanımlanıyor.
  var ns = XmlService.getNamespace('http://base.google.com/ns/1.0');

  // 6. XML'in içindeki 'channel' elemanının içindeki 'item' elemanları alınıyor.
  var items = root.getChildren('channel')[0].getChildren('item');

  // 7. Ürünleri gruplamak için bir harita (dictionary) oluşturuluyor.
  var productMap = {};

  // 8. Her bir ürün için işlem yapılıyor.
  items.forEach(function(item) {
    // 9. Ürün ID'si alınıyor.
    var id = item.getChild('id', ns) ? item.getChild('id', ns).getText() : null;
    if (!id) return;

    // 10. ID'nin ilk kısmı (base kısmı) alınıyor ve bu kısım grup olarak kullanılıyor.
    var baseId = id.split('-')[0];

    // 11. Ürünün bedeni alınıyor, eğer yoksa "Beden Bilgisi Yok" atanıyor.
    var size = item.getChild('size', ns) ? item.getChild('size', ns).getText() : "Beden Bilgisi Yok";

    // 12. Ürünün tipi, custom label bilgileri alınıyor, eğer yoksa uygun varsayılan değerler atanıyor.
    var productType = item.getChild('product_type', ns) ? item.getChild('product_type', ns).getText() : "Ürün Tipi Yok";
    var customLabel1 = item.getChild('custom_label_1', ns) ? item.getChild('custom_label_1', ns).getText() : "Sezon Bilgisi Yok";
    var customLabel2 = item.getChild('custom_label_2', ns) ? item.getChild('custom_label_2', ns).getText() : "Kampanya Bilgisi Yok";
    var customLabel3 = item.getChild('custom_label_3', ns) ? item.getChild('custom_label_3', ns).getText() : "Stok Bilgisi Yok";

    // 13. Eğer bu baseId için daha önce bir ürün eklenmediyse yeni bir ürün ekleniyor.
    if (!productMap[baseId]) {
      productMap[baseId] = {
        title: item.getChild('title').getText(),
        description: item.getChild('description').getText().replace('<![CDATA[', '').replace(']]>', '').trim(),
        link: item.getChild('link').getText(),
        salePrice: item.getChild('sale_price', ns) ? item.getChild('sale_price', ns).getText() : "Fiyat Yok",
        sizes: [],
        productType: productType,
        customLabel1: customLabel1,
        customLabel3: customLabel3,
        customLabel2: customLabel2
      };
    }

    // 14. Ürünün beden bilgisi mevcut ürünün bedenleri listesine ekleniyor.
    productMap[baseId].sizes.push(size);
  });

  // 15. Çıkış satırları için bir dizi oluşturuluyor.
  var outputLines = [];
  
  // 16. Her bir ürün için, ürün bilgileri uygun formatta metin olarak ekleniyor.
  for (var baseId in productMap) {
    var product = productMap[baseId];
    var sizes = product.sizes.join(', ');  // Varyant bedenleri birleştiriliyor

    var line = `[Ürün Tipi: ${product.productType}, Ürün Adı: ${product.title}, ${product.title} ürünün Açıklaması: ${product.description}, ${product.title} ürünün Linki: ${product.link},${product.title} Ürünün Fiyatı: ${product.salePrice}, ${product.title} ürünün Sezon Bilgisi: ${product.customLabel1}, ${product.title} ürünün Stok Bilgisi: ${product.customLabel3}, ${product.title} ürünün Sezon Bilgisi: ${product.customLabel2}, ${product.title} Ürünün Bedenleri: ${sizes}].`;
    outputLines.push(line);
  }

  // 17. Tüm ürün bilgileri tek bir dosyada kaydediliyor.
  var todayDate = Utilities.formatDate(new Date(), Session.getScriptTimeZone(), "yyyyMMdd");
  var fileName = `Perspective_Urunler_${todayDate}.txt`;
  var partContent = outputLines.join("\n");
  var fileBlob = Utilities.newBlob(partContent, 'text/plain', fileName);

  // 18. Dosya bir klasöre kaydediliyor.
  var folderName = `Perspective_Google_Xml_URUNLER_${todayDate}`;
  var folder = DriveApp.createFolder(folderName);
  folder.createFile(fileBlob);
}
