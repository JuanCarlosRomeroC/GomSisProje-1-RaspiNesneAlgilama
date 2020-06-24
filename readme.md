RASPBERRY PI ILE NESNE ALGILAMA
Bu kılavuz, Raspberry Pi'de TensorFlow Lite'ın nasıl kurulacağı ve nesne algılama modellerini çalıştırmak için nasıl kullanılacağı hakkında adım adım talimatlar sağlar.Bu kaynak da aynı isimli EdjeElectronicsin projesinden yararlanılmıştır.

TensorFlow Lite (TFLite) modelleri, Raspberry Pi'deki normal TensorFlow modellerinden çok daha hızlı çalışır.

Bölüm 1 - Raspberry Pi'de TensorFlow Lite Nesne Algılama Modellerinin Ayarlanması ve Çalıştırılması

Raspberry Pi'de TensorFlow Lite kurmak normal TensorFlow'dan çok daha kolay! TensorFlow Lite'ı kurmak için gereken adımlar şunlardır:

1  Raspberry Pi'yi güncelleyin

2  Bu veri havuzunu indirin ve sanal ortam yaratın

3  TensorFlow ve OpenCV'yi yükleyin

4  TensorFlow Lite algılama modelini ayarlama

5  TensorFlow Lite modelini çalıştırın!

Ayrıca bu kılavuzda  bir YouTube videosu hazırladım: https://www.youtube.com/watch?v=H3EL1m2sW1Q
Adım-1  Raspberry Pi'yi güncelleyin;

İlk olarak Raspide bir terminal açıp aşağıdaki kodları yapıştıralım.

sudo apt-get update

sudo apt-get dist-upgrade

Pi'nizi güncellediğinizden bu yana geçen süreye bağlı olarak, güncelleme bir dakika ile bir saat arasında sürebilir.

Güncelleme yapılırken bizde Raspberry konfigürasyon ayarlarından Kamerayı Enable edelim.

Adım 2 Bu veri havuzunu indirin ve sanal ortam yaratın
Ardından, aşağıdaki komutu vererek bu GitHub havuzunu kopyalayın. Depo, TensorFlow Lite'ı çalıştırmak için kullanacağımız komut dosyalarının yanı sıra her şeyi yüklemeyi kolaylaştıracak bir kabuk komut dosyası içerir. 

git clone https://github.com/engbkr/Raspberry-Pi-ile-Nesne-Tanima.git 

Bu, her şeyi Android ve Ahududu Pi'de TensorFlow-Lite-Nesne-Algılama adlı bir klasöre indirir. Çalışması biraz uzun, bu nedenle klasörü "tflite1" olarak yeniden adlandırın ve içine cd yazın:

mv TensorFlow-Lite-Object-Detection-on-Android-and-Raspberry-Pi tflite1

cd tflite1

Rehberin geri kalanı için bu / home / pi / tflite1 dizininde çalışacağız. Sıradaki "tflite1-env" adında bir sanal ortam oluşturmaktır.
Bu kılavuz için sanal bir ortam kullanıyorum, çünkü Pi'nize önceden yüklenmiş olabilecek paket kitaplıklarının sürümleri arasındaki çakışmaları önlüyor. Kendi ortamında kurulu tutmak bu sorunu önlememizi sağlar. Örneğin, TensorFlow v1.8'i Pi'ye diğer kılavuzumu kullanarak zaten yüklediyseniz , bu kurulumu geçersiz kılma konusunda endişelenmeden olduğu gibi bırakabilirsiniz.

Virtualenv'i şu şekilde yükleyin:

sudo pip3 install virtualenv

Ardından, aşağıdakileri düzenleyerek "tflite1-env" sanal ortamını oluşturun:

python3 -m venv tflite1-env

Bu, tflite1 dizininin içinde tflite1-env adlı bir klasör oluşturur. Tflite1-env klasörü, bu ortam için tüm paket kitaplıklarını tutacaktır. Ardından, aşağıdakileri yaparak çevreyi etkinleştirin:

source tflite1-env/bin/activate

source tflite1-env/bin/activate Her yeni terminal penceresi açışınızda ortamı yeniden etkinleştirmek için / home / pi / tflite1 dizininin içinden komutu vermeniz gerekir . Aşağıdaki ekran görüntüsünde gösterildiği gibi, komut isteminizde yoldan önce (tflite1-env) görünüp görünmediğini kontrol ederek ortamın ne zaman etkin olduğunu anlayabilirsiniz.
Bu noktada, eğer sorun çıkarırsanız tflite1 dizininiz aşağıdaki gibi görünmelidir.

Adım 3 TensorFlow Lite bağımlılıklarını ve OpenCV'yi yükleme
Ardından, TensorFlow, OpenCV ve her iki paket için gereken tüm bağımlılıkları yükleyeceğiz. TensorFlow Lite'ı çalıştırmak için OpenCV'ye gerek yoktur, ancak bu depodaki nesne algılama komut dosyaları, görüntüleri yakalamak ve üzerlerine algılama sonuçları çizmek için kullanılır.

İşleri kolaylaştırmak için, tüm paketleri ve bağımlılıkları otomatik olarak indirip yükleyecek bir kabuk komut dosyası Edje Electronics tarafından yazılmıştır. Düzenleyerek çalıştırın.

bash get_pi_requirements.sh

NOT: Kabuk betiği, TensorFlow'un en son sürümünü otomatik olarak yükler. Belirli bir sürümü pip3 install tensorflow==X.XX yüklemek istiyorsanız, komut dosyasını çalıştırdıktan sonra sorun (X.XX yerine yüklemek istediğiniz sürümle değiştirilir). Bu, mevcut yüklemeyi belirtilen sürümle geçersiz kılar
Adım 4 TensorFlow Lite algılama modelini ayarlama
Bir algılama modelinin 2 genel dosyası vardır.İlki dedect.tflite ve ikincisi labelmap.txt dosyasıdır.İstersek kendi modelimizi oluşturabilir yada Google’n hazır olarak sunduğu tflite modeli ile çalışabiliriz.
Google’n örnek TFlite modelini kullanma
Google, MSCOCO veri kümesinden eğitilen ve TensorFlow Lite üzerinde çalışmaya dönüştürülen örnek bir nicelleştirilmiş SSDLite-MobileNet-v2 nesne algılama modeli sağlar. İnsanlar, arabalar, bardaklar vb.Gibi 80 farklı ortak nesneyi tespit edebilir ve tanımlayabilir.

Modeli aşağıdaki kodu girerek indirelim.

wget https://storage.googleapis.com/download.tensorflow.org/models/tflite/coco_ssd_mobilenet_v1_1.0_quant_2018_06_29.zip

Dosyayı arşivden çıkartacak kodu girelim.

unzip coco_ssd_mobilenet_v1_1.0_quant_2018_06_29.zip -d Sample_TFLite_model

Adım 5 TensorFlow Lite modelini çalıştırın!
Modeli çalıştırmadan önce raspi üzerinde açık tüm uygulamaları kapatarak işlemci gücünü modelimizde kullanmalıyız. Her şey tamam uygulamaya geçebiliriz.

python3 TFLite_detection_webcam.py --modeldir=Sample_TFLite_model

Model klasörünüzün "Sample_TFLite_model" adından farklı bir adı varsa, bunun yerine bu adı kullanın. Örneğin, --modeldir=Kendi_Modelim_TFLite_model

