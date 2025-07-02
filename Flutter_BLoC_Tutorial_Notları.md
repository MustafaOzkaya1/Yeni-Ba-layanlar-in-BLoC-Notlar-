# Flutter BLoC Pattern Tutorial - Kapsamlı Öğretici Notlar

## 📋 İçindekiler
1. [Projeye Giriş](#projeye-giriş)
2. [BLoC Pattern Nedir?](#bloc-pattern-nedir)
3. [Proje Yapısı](#proje-yapısı)
4. [Adım Adım Uygulama Geliştirme](#adım-adım-uygulama-geliştirme)
5. [Kodların Detaylı Açıklaması](#kodların-detaylı-açıklaması)
6. [Best Practices ve İpuçları](#best-practices-ve-ipuçları)

---

## 🎯 Projeye Giriş

Bu proje, Flutter'da **BLoC (Business Logic Component)** pattern'ini öğrenmek için tasarlanmış bir **Grocery (Market) Uygulaması**'dır.

### 🛒 Uygulamanın Özellikleri:
- **Ana Sayfa**: Ürünleri listeler
- **Sepet**: Eklenen ürünleri gösterir
- **İstek Listesi**: Beğenilen ürünleri saklar
- **Ürün Yönetimi**: Ürün ekleme/çıkarma işlemleri

### 🏗️ Kullanılan Teknolojiler:
- Flutter 3.x
- flutter_bloc: ^8.x
- bloc: ^8.x

---

## 🧠 BLoC Pattern Nedir?

### Temel Kavramlar:

#### 1. **Event (Olay)**
- Kullanıcı etkileşimlerini temsil eder
- Örnek: "Ürünü sepete ekle", "Sayfa yükle"

#### 2. **State (Durum)**
- Uygulamanın o anki durumunu temsil eder
- Örnek: "Yükleniyor", "Veriler geldi", "Hata oluştu"

#### 3. **BLoC (Business Logic Component)**
- Event'leri alır ve State'leri üretir
- İş mantığının bulunduğu yer

#### 4. **Veri Akışı**
```
UI → Event → BLoC → State → UI
```

---

## 📁 Proje Yapısı

```
lib/
├── data/                          # Veri katmanı
│   ├── cart_items.dart           # Sepet verileri
│   ├── grocery_data.dart         # Ürün verileri
│   └── wishlist_items.dart       # İstek listesi verileri
├── features/                      # Özellik tabanlı yapı
│   ├── home/                     # Ana sayfa özelliği
│   │   ├── bloc/
│   │   │   ├── home_bloc.dart    # Ana iş mantığı
│   │   │   ├── home_event.dart   # Olaylar
│   │   │   └── home_state.dart   # Durumlar
│   │   ├── models/
│   │   │   └── home_product_data_model.dart
│   │   └── ui/
│   │       ├── home.dart         # Ana sayfa UI
│   │       └── product_tile_widget.dart
│   ├── cart/                     # Sepet özelliği
│   │   ├── bloc/
│   │   └── ui/
│   └── wishlist/                 # İstek listesi özelliği
│       ├── bloc/
│       └── ui/
└── main.dart                     # Uygulama başlangıcı
```

---

## 🚀 Adım Adım Uygulama Geliştirme

### Adım 1: Proje Kurulumu

#### 1.1 Bağımlılıkları Ekleme (`pubspec.yaml`)
```yaml
dependencies:
  flutter:
    sdk: flutter
  bloc: ^8.1.2
  flutter_bloc: ^8.1.3
```

#### 1.2 Ana Uygulama Yapısı (`main.dart`)
```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc_tutorial/features/home/ui/home.dart';

void main(){
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData(primaryColor: Colors.teal),
      home: Home(),
    );
  }
}
```

### Adım 2: Veri Modeli Oluşturma

#### 2.1 Ürün Veri Modeli (`home_product_data_model.dart`)
```dart
class ProductDataModel {
  final String id;
  final String name;
  final String description;
  final double price;
  final String imageUrl;
  
  ProductDataModel({
    required this.id,
    required this.name,
    required this.description,
    required this.price,
    required this.imageUrl,
  });
}
```

#### 2.2 Statik Veri Kaynağı (`grocery_data.dart`)
```dart
class GroceryData {
  static List<Map<String, dynamic>> groceryProducts = [
    {
      'id': '1',
      'name': 'Bananas',
      'description': 'Fresh bananas from Ecuador',
      'price': 0.49,
      'imageUrl': 'https://cdn.shopify.com/...',
    },
    // Diğer ürünler...
  ];
}
```

### Adım 3: BLoC Yapısını Kurma

#### 3.1 Event'leri Tanımlama (`home_event.dart`)
```dart
part of 'home_bloc.dart';

@immutable
abstract class HomeEvent {}

// Sayfa ilk açıldığında tetiklenir
class HomeInitialEvent extends HomeEvent {}

// Ürün wishlist'e eklendiğinde
class HomeProductWishlistButtonClickedEvent extends HomeEvent {
  final ProductDataModel clickedProduct;
  HomeProductWishlistButtonClickedEvent({required this.clickedProduct});
}

// Ürün sepete eklendiğinde  
class HomeProductCartButtonClickedEvent extends HomeEvent {
  final ProductDataModel clickedProduct;
  HomeProductCartButtonClickedEvent({required this.clickedProduct});
}

// Navigasyon event'leri
class HomeWishlistButtonNavigateEvent extends HomeEvent {}
class HomeCartButtonNavigateEvent extends HomeEvent {}
```

#### 3.2 State'leri Tanımlama (`home_state.dart`)
```dart
part of 'home_bloc.dart';

@immutable
abstract class HomeState {}

// Action state'ler - UI'da dinlenir ama rebuild yapmaz
abstract class HomeActionState extends HomeState {}

// Normal state'ler - UI rebuild'i tetikler
class HomeInitial extends HomeState {}
class HomeLoadingState extends HomeState {}
class HomeLoadedSuccessState extends HomeState {
  final List<ProductDataModel> products;
  HomeLoadedSuccessState({required this.products});
}
class HomeErrorState extends HomeState {}

// Action state'ler
class HomeNavigateToWishlistPageActionState extends HomeActionState {}
class HomeNavigateToCartPageActionState extends HomeActionState {}
class HomeProductItemWishlistedActionState extends HomeActionState {}
class HomeProductItemCartedActionState extends HomeActionState {}
```

#### 3.3 BLoC Mantığı (`home_bloc.dart`)
```dart
class HomeBloc extends Bloc<HomeEvent, HomeState> {
  HomeBloc() : super(HomeInitial()) {
    // Event handler'ları kaydet
    on<HomeInitialEvent>(homeInitialEvent);
    on<HomeProductWishlistButtonClickedEvent>(homeProductWishlistButtonClickedEvent);
    on<HomeProductCartButtonClickedEvent>(homeProductCartButtonClickedEvent);
    on<HomeWishlistButtonNavigateEvent>(homeWishlistButtonNavigateEvent);
    on<HomeCartButtonNavigateEvent>(homeCartButtonNavigateEvent);
  }

  // Sayfa yüklendiğinde çalışır
  FutureOr<void> homeInitialEvent(HomeInitialEvent event, Emitter<HomeState> emit) async {
    emit(HomeLoadingState()); // Yükleme durumu
    await Future.delayed(Duration(seconds: 3)); // API çağrısı simülasyonu
    
    // Veriyi model'e çevir ve başarı durumu emit et
    emit(HomeLoadedSuccessState(
      products: GroceryData.groceryProducts
          .map((e) => ProductDataModel(
                id: e['id'],
                name: e['name'],
                description: e['description'],
                price: e['price'],
                imageUrl: e['imageUrl']))
          .toList()));
  }

  // Ürünü wishlist'e ekle
  FutureOr<void> homeProductWishlistButtonClickedEvent(
      HomeProductWishlistButtonClickedEvent event, Emitter<HomeState> emit) {
    wishlistItems.add(event.clickedProduct);
    emit(HomeProductItemWishlistedActionState());
  }

  // Ürünü sepete ekle
  FutureOr<void> homeProductCartButtonClickedEvent(
      HomeProductCartButtonClickedEvent event, Emitter<HomeState> emit) {
    cartItems.add(event.clickedProduct);
    emit(HomeProductItemCartedActionState());
  }

  // Navigasyon işlemleri
  FutureOr<void> homeWishlistButtonNavigateEvent(
      HomeWishlistButtonNavigateEvent event, Emitter<HomeState> emit) {
    emit(HomeNavigateToWishlistPageActionState());
  }

  FutureOr<void> homeCartButtonNavigateEvent(
      HomeCartButtonNavigateEvent event, Emitter<HomeState> emit) {
    emit(HomeNavigateToCartPageActionState());
  }
}
```

### Adım 4: UI Geliştirme

#### 4.1 Ana Sayfa UI'ı (`home.dart`)
```dart
class Home extends StatefulWidget {
  @override
  State<Home> createState() => _HomeState();
}

class _HomeState extends State<Home> {
  final HomeBloc homeBloc = HomeBloc();

  @override
  void initState() {
    homeBloc.add(HomeInitialEvent()); // Sayfa açılınca veri yükle
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return BlocConsumer<HomeBloc, HomeState>(
      bloc: homeBloc,
      // Action state'leri dinle ama rebuild yapma
      listenWhen: (previous, current) => current is HomeActionState,
      // Normal state'lerde rebuild yap
      buildWhen: (previous, current) => current is! HomeActionState,
      
      // Action state'leri handle et
      listener: (context, state) {
        if (state is HomeNavigateToCartPageActionState) {
          Navigator.push(context, MaterialPageRoute(builder: (context) => Cart()));
        } else if (state is HomeNavigateToWishlistPageActionState) {
          Navigator.push(context, MaterialPageRoute(builder: (context) => Wishlist()));
        } else if (state is HomeProductItemCartedActionState) {
          ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('Item Carted')));
        } else if (state is HomeProductItemWishlistedActionState) {
          ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('Item Wishlisted')));
        }
      },
      
      // UI'ı build et
      builder: (context, state) {
        switch (state.runtimeType) {
          case HomeLoadingState:
            return Scaffold(body: Center(child: CircularProgressIndicator()));
            
          case HomeLoadedSuccessState:
            final successState = state as HomeLoadedSuccessState;
            return Scaffold(
              appBar: AppBar(
                backgroundColor: Colors.teal,
                title: Text('Grocery App'),
                actions: [
                  IconButton(
                    onPressed: () => homeBloc.add(HomeWishlistButtonNavigateEvent()),
                    icon: Icon(Icons.favorite_border)),
                  IconButton(
                    onPressed: () => homeBloc.add(HomeCartButtonNavigateEvent()),
                    icon: Icon(Icons.shopping_bag_outlined)),
                ],
              ),
              body: ListView.builder(
                itemCount: successState.products.length,
                itemBuilder: (context, index) {
                  return ProductTileWidget(
                    homeBloc: homeBloc,
                    productDataModel: successState.products[index]);
                }),
            );
            
          case HomeErrorState:
            return Scaffold(body: Center(child: Text('Error')));
            
          default:
            return SizedBox();
        }
      },
    );
  }
}
```

#### 4.2 Ürün Kartı Widget'ı (`product_tile_widget.dart`)
```dart
class ProductTileWidget extends StatelessWidget {
  final ProductDataModel productDataModel;
  final HomeBloc homeBloc;
  
  const ProductTileWidget({
    required this.productDataModel, 
    required this.homeBloc
  });

  @override
  Widget build(BuildContext context) {
    return Container(
      margin: const EdgeInsets.all(10),
      padding: const EdgeInsets.all(10),
      decoration: BoxDecoration(
        borderRadius: BorderRadius.circular(10),
        border: Border.all(color: Colors.black)
      ),
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: [
          // Ürün resmi
          Container(
            height: 200,
            width: double.maxFinite,
            decoration: BoxDecoration(
              image: DecorationImage(
                fit: BoxFit.cover,
                image: NetworkImage(productDataModel.imageUrl)
              )
            ),
          ),
          SizedBox(height: 20),
          
          // Ürün bilgileri
          Text(productDataModel.name, 
               style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
          Text(productDataModel.description),
          SizedBox(height: 20),
          
          // Fiyat ve aksiyonlar
          Row(
            mainAxisAlignment: MainAxisAlignment.spaceBetween,
            children: [
              Text("\$${productDataModel.price}", 
                   style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
              Row(
                children: [
                  // Wishlist butonu
                  IconButton(
                    onPressed: () {
                      homeBloc.add(HomeProductWishlistButtonClickedEvent(
                        clickedProduct: productDataModel));
                    },
                    icon: Icon(Icons.favorite_border)
                  ),
                  // Sepet butonu
                  IconButton(
                    onPressed: () {
                      homeBloc.add(HomeProductCartButtonClickedEvent(
                        clickedProduct: productDataModel));
                    },
                    icon: Icon(Icons.shopping_bag_outlined)
                  ),
                ],
              )
            ],
          ),
        ],
      ),
    );
  }
}
```

---

## 🔍 Kodların Detaylı Açıklaması

### BloC Consumer vs Builder vs Listener

#### 1. **BlocConsumer**
- Hem dinleme hem de UI build etme işlerini yapar
- `listenWhen` ve `buildWhen` ile kontrolü sağlar

#### 2. **BlocBuilder**
- Sadece UI build eder
- State değişikliklerinde widget'ı yeniden çizer

#### 3. **BlocListener**
- Sadece dinler, UI build etmez
- Navigasyon, snackbar gibi yan etkiler için kullanılır

### Event-State Eşleştirmesi

```dart
// Event tetiklenir
homeBloc.add(HomeProductCartButtonClickedEvent(clickedProduct: product));

// BLoC işler
FutureOr<void> homeProductCartButtonClickedEvent(...) {
    cartItems.add(event.clickedProduct);
    emit(HomeProductItemCartedActionState()); // State yayınla
}

// UI dinler ve tepki verir
listener: (context, state) {
    if (state is HomeProductItemCartedActionState) {
        ScaffoldMessenger.of(context).showSnackBar(SnackBar(content: Text('Item Carted')));
    }
}
```

### Global State Yönetimi

```dart
// Global listeler - Basit state management için
List<ProductDataModel> cartItems = [];
List<ProductDataModel> wishlistItems = [];
```

---

## 💡 Best Practices ve İpuçları

### 1. **Klasör Yapısı**
- Feature-based organization kullanın
- Her feature kendi bloc/ui/model klasörlerine sahip olsun

### 2. **Event Adlandırma**
```dart
// ❌ Kötü
class ButtonClicked extends HomeEvent {}

// ✅ İyi  
class HomeProductCartButtonClickedEvent extends HomeEvent {}
```

### 3. **State Types**
```dart
// UI rebuild için
abstract class HomeState {}

// Yan etkiler için (navigation, snackbar)
abstract class HomeActionState extends HomeState {}
```

### 4. **BlocConsumer Kullanımı**
```dart
BlocConsumer<HomeBloc, HomeState>(
  // Action state'leri sadece dinle
  listenWhen: (previous, current) => current is HomeActionState,
  // Normal state'lerde rebuild yap
  buildWhen: (previous, current) => current is! HomeActionState,
  // ...
)
```

### 5. **Memory Leaks'ten Kaçınma**
```dart
@override
void dispose() {
  homeBloc.close(); // BLoC'u kapat
  super.dispose();
}
```

### 6. **Error Handling**
```dart
try {
  // API çağrısı
  final data = await apiService.getProducts();
  emit(HomeLoadedSuccessState(products: data));
} catch (e) {
  emit(HomeErrorState(message: e.toString()));
}
```

### 7. **Testing**
```dart
// BLoC test etmek kolay
blocTest<HomeBloc, HomeState>(
  'should emit loading then success when data is loaded',
  build: () => HomeBloc(),
  act: (bloc) => bloc.add(HomeInitialEvent()),
  expect: () => [HomeLoadingState(), HomeLoadedSuccessState(products: [])],
);
```

---

## 🎯 Özet

Bu proje ile öğrendikleriniz:

1. **BLoC Pattern'in Temelleri**: Event-State-BLoC üçgeni
2. **Flutter BLoC Kütüphanesi**: BlocConsumer, BlocBuilder, BlocListener
3. **Feature-Based Architecture**: Modüler ve ölçeklenebilir kod yapısı
4. **State Management**: Global state yönetimi teknikleri
5. **UI-Business Logic Ayrımı**: Temiz mimari prensipleri

### Sonraki Adımlar:
- Gerçek API entegrasyonu
- Unit ve widget testleri
- Hata yönetimi geliştirme
- Performans optimizasyonları
- Daha kompleks state senaryoları

Bu yapı ile büyük ölçekli Flutter uygulamaları geliştirebilirsiniz! 🚀 