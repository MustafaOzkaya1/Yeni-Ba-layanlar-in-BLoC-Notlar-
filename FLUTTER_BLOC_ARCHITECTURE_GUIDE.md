# Flutter BLoC Architecture - Kapsamlı Kullanım Klavuzu

## İçindekiler
1. [Proje Genel Bakış](#proje-genel-bakış)
2. [Mimari Yapı](#mimari-yapı)
3. [Klasör Yapısı](#klasör-yapısı)
4. [Temel Teknolojiler](#temel-teknolojiler)
5. [Detaylı Implementasyon](#detaylı-implementasyon)
6. [Kod Örnekleri](#kod-örnekleri)
7. [Kurulum ve Kullanım](#kurulum-ve-kullanım)
8. [En İyi Pratikler](#en-iyi-pratikler)
9. [Test Stratejisi](#test-stratejisi)

## Proje Genel Bakış

Bu proje, **Clean Architecture** prensipleri ile **BLoC Pattern** kullanarak geliştirilmiş modern bir Flutter uygulamasıdır. Firebase backend servisleri ile entegre edilmiş olan proje, bütçe yönetimi için tasarlanmıştır.

### Ana Özellikler
- ✅ Clean Architecture mimarisi
- ✅ BLoC State Management
- ✅ Dependency Injection (GetIt)
- ✅ Repository Pattern
- ✅ Use Case Pattern
- ✅ Result/Either Pattern (Error Handling)
- ✅ Firebase Integration (Firestore, Auth)
- ✅ Unit Testing (Mocktail)
- ✅ Mixin Pattern
- ✅ Extension Methods
- ✅ Feature-based Modüler Yapı

## Mimari Yapı

### Clean Architecture Katmanları

```
┌─────────────────────────────────────────┐
│              Presentation               │
│           (UI, BLoC, Views)            │
├─────────────────────────────────────────┤
│               Domain                    │
│        (Use Cases, Entities,           │
│         Repository Interfaces)          │
├─────────────────────────────────────────┤
│               Data                      │
│      (Repository Implementations,       │
│        Data Sources, Models)           │
└─────────────────────────────────────────┘
```

### BLoC Pattern Akışı

```
UI Event → BLoC → Use Case → Repository → Data Source → Firebase
                    ↓
UI State ← BLoC ← Repository ← Data Source ← Firebase Response
```

## Klasör Yapısı

```
lib/
├── core/                          # Çekirdek bileşenler
│   ├── components/               # Yeniden kullanılabilir UI bileşenleri
│   ├── constants/               # Uygulama sabitleri
│   ├── extension/               # Extension metodları
│   ├── params/                  # Use case parametreleri
│   └── result/                  # Result/Either pattern
├── features/                    # Feature-based modüller
│   ├── auth/                   # Kimlik doğrulama
│   │   ├── data/
│   │   │   ├── data_source/   # Veri kaynakları
│   │   │   └── repo/          # Repository implementasyonları
│   │   ├── domain/
│   │   │   ├── repo/          # Repository interface'leri
│   │   │   └── use_cases/     # İş mantığı
│   │   └── presentation/
│   │       ├── bloc/          # BLoC state management
│   │       ├── components/    # Feature-specific UI bileşenleri
│   │       └── view/          # Sayfalar ve mixin'ler
│   ├── feeds/                 # Ana akış modülü
│   ├── home/                  # Ana sayfa
│   ├── profile/               # Profil yönetimi
│   └── search/                # Arama ve filtreleme
├── product/                   # Ürün-specific bileşenler
│   ├── appToast/             # Toast bildirimleri
│   ├── enum/                 # Enum tanımları
│   ├── exception/            # Hata yönetimi
│   ├── globals.dart          # Global değişkenler
│   ├── injection/            # Dependency injection
│   ├── popup/                # Popup bileşenleri
│   └── theme/                # Tema ve stil
└── main.dart                 # Uygulama giriş noktası
```

## Temel Teknolojiler

### Bağımlılıklar (pubspec.yaml)

```yaml
dependencies:
  flutter:
    sdk: flutter
  
  # State Management
  flutter_bloc: ^7.3.3
  equatable: ^2.0.5
  
  # Dependency Injection
  get_it: ^7.7.0
  
  # Functional Programming
  dartz: ^0.10.1
  
  # Firebase
  firebase_core: ^2.31.0
  firebase_auth: ^4.19.5
  cloud_firestore: ^4.17.3
  
  # UI Components
  flutter_svg: ^2.0.10+1
  curved_navigation_bar: ^1.0.3
  
  # Utilities
  intl: ^0.19.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.0
  mocktail: ^1.0.3
```

## Detaylı Implementasyon

### 1. BLoC State Management

#### Bloc Yapısı
```dart
// auth_bloc.dart
class AuthBloc extends Bloc<AuthEvent, AuthState> {
  AuthBloc({
    required SignedInUseCase signedInUseCase,
  }) : _signedInUseCase = signedInUseCase,
       super(AuthInitial()) {
    on<SignInEvent>(_signInHandler);
    on<TogglePasswordVisibilityEvent>(_togglePasswordVisibilityHandler);
  }

  final SignedInUseCase _signedInUseCase;

  Future<void> _signInHandler(
    SignInEvent event,
    Emitter<AuthState> emit,
  ) async {
    final result = await _signedInUseCase(
      UserParams(userName: event.username, password: event.password),
    );
    
    result.fold(
      (error) => emit(AuthError(message: error.message)),
      (user) {
        globals.currentUser = user;
        emit(AuthSignedIn());
      },
    );
  }
}
```

#### Event Tanımları
```dart
// auth_event.dart
sealed class AuthEvent extends Equatable {
  const AuthEvent();
  @override
  List<Object> get props => [];
}

final class SignInEvent extends AuthEvent {
  const SignInEvent({required this.username, required this.password});
  
  final String username;
  final String password;

  @override
  List<Object> get props => [username, password];
}

final class TogglePasswordVisibilityEvent extends AuthEvent {}
```

#### State Tanımları
```dart
// auth_state.dart
sealed class AuthState extends Equatable {
  const AuthState();
  @override
  List<Object?> get props => [];
}

final class AuthInitial extends AuthState {}

final class AuthError extends AuthState {
  final String? message;
  const AuthError({required this.message});
  
  @override
  List<Object?> get props => [message];
}

final class AuthSignedIn extends AuthState {}

final class IsPasswordVisible extends AuthState {
  final bool isPasswordVisible;
  const IsPasswordVisible({required this.isPasswordVisible});
  
  @override
  List<Object?> get props => [isPasswordVisible];
}
```

### 2. Use Case Pattern

#### Base Use Case Classes
```dart
// use_case_params.dart
abstract class UsecaseWithParams<Type, Params> {
  const UsecaseWithParams();
  ResultFuture<Type> call(Params params);
}

abstract class UsecaseWithoutParams<Type> {
  const UsecaseWithoutParams();
  ResultFuture<Type> call();
}
```

#### Konkret Use Case Implementasyonu
```dart
// signed_in_use_case.dart
class SignedInUseCase extends UsecaseWithParams<UserModel, UserParams> {
  const SignedInUseCase(this._repo);
  
  final SignInRepo _repo;

  @override
  ResultFuture<UserModel> call(UserParams params) =>
      _repo.signIn(username: params.userName, password: params.password);
}
```

### 3. Repository Pattern

#### Repository Interface
```dart
// sign_in_repo.dart
abstract class SignInRepo {
  const SignInRepo();
  
  ResultFuture<UserModel> signIn({
    required String username,
    required String password,
  });
}
```

#### Repository Implementation
```dart
// sign_in_repo_impl.dart
class SignInRepoImpl implements SignInRepo {
  SignInRepoImpl({required SignInDataSource remoteDataSource})
      : _remoteDataSource = remoteDataSource;
      
  final SignInDataSource _remoteDataSource;

  @override
  ResultFuture<UserModel> signIn({
    required String username,
    required String password,
  }) async {
    try {
      final result = await _remoteDataSource.signIn(
        username: username,
        password: password,
      );
      
      if (result == null) {
        throw const ServerException(
          message: 'User not found',
          statusCode: 404,
        );
      }
      
      return Right(result);
    } on ServerException catch (e) {
      return Left(ServerFailure(
        message: e.message,
        statusCode: e.statusCode,
      ));
    }
  }
}
```

### 4. Data Source Pattern

#### Data Source Interface
```dart
// sign_in_data_source.dart
abstract class SignInDataSource {
  const SignInDataSource();
  
  Future<UserModel?> signIn({
    required String username,
    required String password,
  });
}
```

#### Firebase Data Source Implementation
```dart
// sign_in_data_source_impl.dart
class SignInDataSourceImpl implements SignInDataSource {
  @override
  Future<UserModel?> signIn({
    required String username,
    required String password,
  }) async {
    try {
      QuerySnapshot userSnapshot = await FirebaseFirestore.instance
          .collection('Tbl_Users')
          .where('UserName', isEqualTo: username)
          .where('Password', isEqualTo: password)
          .get();

      if (userSnapshot.docs.isNotEmpty) {
        DocumentSnapshot userDoc = userSnapshot.docs.first;
        return UserModel.fromJson(userDoc.data() as Map<String, dynamic>);
      }
      
      return null;
    } catch (e) {
      return null;
    }
  }
}
```

### 5. Result/Either Pattern (Error Handling)

#### Result Type Definitions
```dart
// result.dart
typedef Result<T> = Either<Failure, T>;

// result_future.dart
typedef ResultFuture<T> = Future<Either<Failure, T>>;
typedef Result<T> = Either<Failure, T>;
```

#### Exception Classes
```dart
// server_exception.dart
class ServerException extends Equatable implements Exception {
  const ServerException({required this.message, required this.statusCode});

  final String message;
  final int statusCode;

  @override
  List<dynamic> get props => [message, statusCode];
}

// server_failure.dart
abstract class Failure extends Equatable {
  const Failure({required this.statusCode, this.message});

  final String? message;
  final int? statusCode;

  @override
  List<dynamic> get props => [message, statusCode];
}

class ServerFailure extends Failure {
  const ServerFailure({
    required String message,
    required int statusCode,
  }) : super(message: message, statusCode: statusCode);

  ServerFailure.fromException(ServerException exception)
      : this(message: exception.message, statusCode: exception.statusCode);
}
```

### 6. Dependency Injection (GetIt)

#### Main Container
```dart
// _injection_container_main.dart
final sl = GetIt.instance;

class InjectionContainer {
  InjectionContainer._();
  
  static InjectionContainer? _instance;
  static InjectionContainer get instance =>
      _instance ??= InjectionContainer._();

  Future<void> init() async {
    _initSinIn();
    _initBudget();
    _initBudgetStatus();
    _initGetUser();
  }
}
```

#### Feature-specific Injection
```dart
// auth_injection.dart
void _initSinIn() {
  sl
    ..registerLazySingleton<SignInDataSource>(
      () => SignInDataSourceImpl(),
    )
    ..registerLazySingleton<SignInRepo>(
      () => SignInRepoImpl(remoteDataSource: sl()),
    )
    ..registerLazySingleton(() => SignedInUseCase(sl()))
    ..registerFactory(
      () => AuthBloc(signedInUseCase: sl()),
    );
}
```

### 7. Model Classes

```dart
// user_model.dart
class UserModel {
  final String displayName;
  final String password;
  final String unitName;
  final int userId;
  final String userName;

  UserModel({
    required this.displayName,
    required this.password,
    required this.unitName,
    required this.userId,
    required this.userName,
  });

  factory UserModel.fromJson(Map<String, dynamic> json) {
    return UserModel(
      displayName: json['DisplayName'] ?? '',
      password: json['Password'] ?? '',
      unitName: json['UnitName'] ?? '',
      userId: json['UserId'] ?? -1,
      userName: json['UserName'] ?? '',
    );
  }
}
```

### 8. Mixin Pattern

```dart
// sign_in_mixin.dart
mixin SignInMixin on State<SignIn> {
  late final AuthBloc _authBloc;
  final TextEditingController _usernameController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();

  @override
  void initState() {
    _authBloc = sl<AuthBloc>();
    super.initState();
  }
}

// sign_in.dart
class _SignInState extends State<SignIn> with SignInMixin {
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => _authBloc,
      child: Scaffold(
        body: BlocListener<AuthBloc, AuthState>(
          listener: (context, state) {
            if (state is AuthError) {
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(
                  content: Text('Kullanıcı adı veya parola hatalı'),
                  backgroundColor: AppColors.errorColor,
                ),
              );
            } else if (state is AuthSignedIn) {
              Navigator.push(
                context,
                MaterialPageRoute(builder: (context) => const Home()),
              );
            }
          },
          child: _buildBody(),
        ),
      ),
    );
  }
}
```

### 9. Extension Methods

```dart
// context_extension.dart
extension ContextExtension on BuildContext {
  double get height => MediaQuery.of(this).size.height;
  double get width => MediaQuery.of(this).size.width;
  double get topPadding => MediaQuery.of(this).padding.top;
  double get bottomPadding => MediaQuery.of(this).padding.bottom;

  double dynamicHeight(double value) =>
      (height - topPadding - bottomPadding) / AppConst.designHeight * value;

  double dynamicWidth(double value) => width / AppConst.designWidth * value;
}
```

### 10. Theme System

```dart
// color_theme.dart
class AppColors {
  static const primaryColor = Color(0xFF223244);
  static const lightBackground = Color(0xFFFFFFFC);
  static const orange = Color(0xFFFE6635);
  static const successColor = Color(0xFF50B240);
  static const errorColor = Color(0xFFFF453A);
  static const dividerColor = Color(0xFFD9D9D9);
  // ... diğer renkler
}

// text_theme.dart
class TextThemeLight {
  TextStyle get titleSignInLarge => TextStyle(
    fontFamily: 'PoppinsBold',
    fontSize: 24,
    fontWeight: FontWeight.bold,
    color: AppColors.dark,
  );
  
  InputDecoration get inputDecoration => InputDecoration(
    border: InputBorder.none,
    contentPadding: EdgeInsets.symmetric(horizontal: 16, vertical: 12),
    labelStyle: TextStyle(color: AppColors.hintText),
  );
}
```

## Kurulum ve Kullanım

### 1. Projeyi Klonlama ve Kurulum

```bash
# Projeyi klonla
git clone <repository-url>
cd budget_balance_mobile

# Bağımlılıkları yükle
flutter pub get

# Firebase kurulumu
flutterfire configure

# Projeyi çalıştır
flutter run
```

### 2. Firebase Konfigürasyonu

```bash
# Firebase CLI kurulumu
npm install -g firebase-tools

# Firebase login
firebase login

# FlutterFire CLI kurulumu
dart pub global activate flutterfire_cli

# Firebase projeyi yapılandırma
flutterfire configure
```

### 3. Yeni Feature Ekleme Adımları

#### A. Klasör Yapısını Oluştur
```
lib/features/new_feature/
├── data/
│   ├── data_sources/
│   ├── models/
│   └── repo/
├── domain/
│   ├── repo/
│   └── use_cases/
└── presentation/
    ├── bloc/
    ├── components/
    └── view/
```

#### B. Domain Katmanı
```dart
// 1. Entity/Model oluştur
class NewFeatureModel {
  final String id;
  final String name;
  
  NewFeatureModel({required this.id, required this.name});
  
  factory NewFeatureModel.fromJson(Map<String, dynamic> json) {
    return NewFeatureModel(
      id: json['id'] ?? '',
      name: json['name'] ?? '',
    );
  }
}

// 2. Repository interface oluştur
abstract class NewFeatureRepo {
  ResultFuture<List<NewFeatureModel>> getItems();
  ResultFuture<void> addItem(NewFeatureModel item);
}

// 3. Use case oluştur
class GetNewFeatureItemsUseCase extends UsecaseWithoutParams<List<NewFeatureModel>> {
  const GetNewFeatureItemsUseCase(this._repo);
  final NewFeatureRepo _repo;

  @override
  ResultFuture<List<NewFeatureModel>> call() => _repo.getItems();
}
```

#### C. Data Katmanı
```dart
// 1. Data source interface
abstract class NewFeatureDataSource {
  Future<List<NewFeatureModel>> getItems();
}

// 2. Data source implementation
class NewFeatureDataSourceImpl implements NewFeatureDataSource {
  @override
  Future<List<NewFeatureModel>> getItems() async {
    // Firebase veya API çağrısı
    final snapshot = await FirebaseFirestore.instance
        .collection('new_feature_items')
        .get();
    
    return snapshot.docs
        .map((doc) => NewFeatureModel.fromJson(doc.data()))
        .toList();
  }
}

// 3. Repository implementation
class NewFeatureRepoImpl implements NewFeatureRepo {
  const NewFeatureRepoImpl({required this.dataSource});
  final NewFeatureDataSource dataSource;

  @override
  ResultFuture<List<NewFeatureModel>> getItems() async {
    try {
      final result = await dataSource.getItems();
      return Right(result);
    } on ServerException catch (e) {
      return Left(ServerFailure.fromException(e));
    }
  }
}
```

#### D. Presentation Katmanı
```dart
// 1. Events
sealed class NewFeatureEvent extends Equatable {
  const NewFeatureEvent();
  @override
  List<Object> get props => [];
}

class GetItemsEvent extends NewFeatureEvent {}

// 2. States
sealed class NewFeatureState extends Equatable {
  const NewFeatureState();
  @override
  List<Object> get props => [];
}

class NewFeatureInitial extends NewFeatureState {}
class NewFeatureLoading extends NewFeatureState {}
class NewFeatureLoaded extends NewFeatureState {
  final List<NewFeatureModel> items;
  const NewFeatureLoaded({required this.items});
  
  @override
  List<Object> get props => [items];
}

// 3. BLoC
class NewFeatureBloc extends Bloc<NewFeatureEvent, NewFeatureState> {
  NewFeatureBloc({required this.getItemsUseCase}) : super(NewFeatureInitial()) {
    on<GetItemsEvent>(_getItemsHandler);
  }

  final GetNewFeatureItemsUseCase getItemsUseCase;

  Future<void> _getItemsHandler(
    GetItemsEvent event,
    Emitter<NewFeatureState> emit,
  ) async {
    emit(NewFeatureLoading());
    
    final result = await getItemsUseCase();
    
    result.fold(
      (failure) => emit(NewFeatureError(message: failure.message)),
      (items) => emit(NewFeatureLoaded(items: items)),
    );
  }
}
```

#### E. Dependency Injection
```dart
// new_feature_injection.dart
void _initNewFeature() {
  sl
    ..registerLazySingleton<NewFeatureDataSource>(
      () => NewFeatureDataSourceImpl(),
    )
    ..registerLazySingleton<NewFeatureRepo>(
      () => NewFeatureRepoImpl(dataSource: sl()),
    )
    ..registerLazySingleton(() => GetNewFeatureItemsUseCase(sl()))
    ..registerFactory(
      () => NewFeatureBloc(getItemsUseCase: sl()),
    );
}

// _injection_container_main.dart içinde
Future<void> init() async {
  _initSinIn();
  _initBudget();
  _initBudgetStatus();
  _initGetUser();
  _initNewFeature(); // Yeni eklenen
}
```

## En İyi Pratikler

### 1. BLoC Prensipleri
- Her event için ayrı handler metodu oluşturun
- State'leri immutable tutun
- BlocBuilder ve BlocListener'ı doğru kullanın
- buildWhen ve listenWhen parametrelerini optimize edin

### 2. Clean Architecture
- Dependency rule'unu ihlal etmeyin (içerideki katmanlar dışarıdakileri bilmemeli)
- Her katmanın sorumluluğunu net ayrın
- Interface'leri domain katmanında tanımlayın

### 3. Error Handling
- Try-catch blokları sadece data katmanında kullanın
- Either pattern ile hataları type-safe şekilde yönetin
- Kullanıcı dostu hata mesajları gösterin

### 4. Testing
- Her use case için unit test yazın
- Repository'ler için mock kullanın
- BLoC'lar için integration test yazın

### 5. Code Organization
- Feature-based klasör yapısını koruyun
- Naming convention'ları tutarlı kullanın
- Barrel exports kullanarak import'ları temizleyin

## Test Stratejisi

### Unit Test Örneği
```dart
// sign_in_test.dart
void main() {
  late SignedInUseCase usecase;
  late SignInRepo repo;
  
  setUpAll(() {
    repo = SignInRepoMock();
    usecase = SignedInUseCase(repo);
  });

  const UserParams params = UserParams(
    userName: 'testuser',
    password: 'password123',
  );

  final UserModel user = UserModel(
    userId: 1,
    displayName: 'Test User',
    password: 'password123',
    unitName: 'Test Unit',
    userName: 'testuser',
  );

  test(
    'should call the [SignInRepo.signIn] when correct params are passed',
    () async {
      // Arrange
      when(() => repo.signIn(
        username: params.userName,
        password: params.password,
      )).thenAnswer((_) async => Right(user));

      // Act
      final Result<UserModel> result = await usecase(params);

      // Assert
      expect(result, equals(Right<dynamic, UserModel>(user)));
      verify(() => repo.signIn(
        username: params.userName,
        password: params.password,
      )).called(1);
      verifyNoMoreInteractions(repo);
    },
  );
}
```

### Mock Repository
```dart
// sign_in_repo_mock.dart
class SignInRepoMock extends Mock implements SignInRepo {}
```

### BLoC Test
```dart
void main() {
  late AuthBloc authBloc;
  late SignedInUseCase signedInUseCase;

  setUpAll(() {
    signedInUseCase = MockSignedInUseCase();
    authBloc = AuthBloc(signedInUseCase: signedInUseCase);
  });

  blocTest<AuthBloc, AuthState>(
    'emits [AuthSignedIn] when SignInEvent is successful',
    build: () {
      when(() => signedInUseCase(any()))
          .thenAnswer((_) async => Right(mockUser));
      return authBloc;
    },
    act: (bloc) => bloc.add(
      SignInEvent(username: 'test', password: 'test'),
    ),
    expect: () => [AuthSignedIn()],
  );
}
```

## Sonuç

Bu kapsamlı kullanım klavuzu, modern Flutter uygulaması geliştirirken Clean Architecture ve BLoC pattern'inin en etkili şekilde nasıl kullanılacağını göstermektedir. 

### Önemli Noktalar:
1. **Separation of Concerns**: Her katmanın sorumluluğu net şekilde ayrılmıştır
2. **Testability**: Tüm bileşenler kolayca test edilebilir
3. **Maintainability**: Kod bakımı ve genişletilmesi kolaydır
4. **Scalability**: Yeni feature'lar kolayca eklenebilir
5. **Type Safety**: Dart'ın type system'i maksimum şekilde kullanılmıştır

Bu kılavuzu takip ederek, tutarlı, ölçeklenebilir ve maintainable Flutter uygulamaları geliştirebilirsiniz. 