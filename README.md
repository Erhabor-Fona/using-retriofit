# 🚀 Making a Request to a New API Endpoint Using Retrofit in Flutter

This guide provides a **step-by-step implementation** of making an API request using **Retrofit**, **DataManager**, and **Bloc** in Flutter.

---

## 📌 Steps Overview:
1. Define the Request Model
2. Define the Response Model
3. Add a New Method in Retrofit API Client
4. Implement the Method in DataManager
5. Implement the Bloc to Manage API Calls
6. Call the API from the UI

---

## 📝 Step 1: Define the Request Model
Create `new_feature_request_body.dart` in `dataProvider/model/requestBody/`.

```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'new_feature_request_body.freezed.dart';
part 'new_feature_request_body.g.dart';

@freezed
class NewFeatureRequestBody with _$NewFeatureRequestBody {
  const factory NewFeatureRequestBody({
    required String param1,
    required int param2,
    bool? optionalParam,
  }) = _NewFeatureRequestBody;

  factory NewFeatureRequestBody.fromJson(Map<String, dynamic> json) =>
      _$NewFeatureRequestBodyFromJson(json);
}
```

✅ Ensures the request data is correctly structured, the run the following command:
```
flutter pub run build_runner build --delete-conflicting-outputs
```


## 📌 Step 2: Define the Response Model
Create `new_feature_response.dart` in `dataProvider/model/response/`.
```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'new_feature_response.freezed.dart';
part 'new_feature_response.g.dart';

@freezed
class NewFeatureResponse with _$NewFeatureResponse {
  const factory NewFeatureResponse({
    required bool success,
    required String message,
    Map<String, dynamic>? data,
  }) = _NewFeatureResponse;

  factory NewFeatureResponse.fromJson(Map<String, dynamic> json) =>
      _$NewFeatureResponseFromJson(json);
}
```
✅ This ensures the response is properly parsed when received.
run:
```
flutter pub run build_runner build --delete-conflicting-outputs
```

## 🌐 Step 3: Add a New Method in Retrofit API Client

Modify `network_api_client.dart` to include the new API request.

```dart
import 'package:retrofit/retrofit.dart';
import 'package:dio/dio.dart';
import '../model/requestBody/new_feature_request_body.dart';
import '../model/response/new_feature_response.dart';

part 'network_api_client.g.dart';

@RestApi(baseUrl: "http://your-api-url.com/api/v1/")
abstract class NetworkApiClient {
  factory NetworkApiClient(Dio dio, {String baseUrl}) = _NetworkApiClient;

  @POST("new-endpoint")
  Future<NewFeatureResponse> newFeatureRequest(@Body() NewFeatureRequestBody body);
}
```
After which run:
```
flutter pub run build_runner build --delete-conflicting-outputs
```
✅ Retrofit now knows how to call the new endpoint.

## 🛠 Step 4: Implement the Method in DataManager

Modify data_manager.dart to call the new API from NetworkApiClient.
```dart
import '../dataProvider/model/requestBody/new_feature_request_body.dart';
import '../dataProvider/model/response/new_feature_response.dart';

class DataManager implements DataManagerImp {
  late final NetworkApiClient _networkApiClient;

  DataManager({required NetworkApiClient networkApiClient}) {
    _networkApiClient = networkApiClient;
  }

  @override
  Future<NewFeatureResponse> newFeature(NewFeatureRequestBody body) {
    return _networkApiClient.newFeatureRequest(body).catchError((e) {
      Utils.console("API Error: $e");
      throw Exception("Request failed: $e");
    });
  }
}
```
✅ This connects the API client to DataManager, making it reusable across the app.
## ⚡ Step 5: Create the Bloc for API Calls
Create ```new_feature_cubit.dart``` to handle API requests.

```dart
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../dataProvider/model/requestBody/new_feature_request_body.dart';
import '../../dataProvider/model/response/new_feature_response.dart';
import '../../dataManager/data_manager.dart';

class NewFeatureCubit extends Cubit<NewFeatureState> {
  final DataManager _dataManager;

  NewFeatureCubit(this._dataManager) : super(NewFeatureInitial());

  Future<void> fetchNewFeatureData(String param1, int param2) async {
    emit(NewFeatureLoading());
    try {
      final response = await _dataManager.newFeature(
        NewFeatureRequestBody(param1: param1, param2: param2),
      );
      emit(NewFeatureSuccess(response));
    } catch (e) {
      emit(NewFeatureFailure("Failed to load data"));
    }
  }
}
```
✅ The Bloc ensures the API call is managed properly.
## 📲 Step 6: Call the API from the UI
Modify your UI screen to trigger the API call.
```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../bloc/new_feature_cubit.dart';

class NewFeatureScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("New Feature")),
      body: BlocConsumer<NewFeatureCubit, NewFeatureState>(
        listener: (context, state) {
          if (state is NewFeatureFailure) {
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(content: Text(state.message)),
            );
          }
        },
        builder: (context, state) {
          if (state is NewFeatureLoading) {
            return Center(child: CircularProgressIndicator());
          } else if (state is NewFeatureSuccess) {
            return Center(child: Text("Success: ${state.response.message}"));
          }
          return Center(
            child: ElevatedButton(
              onPressed: () {
                context.read<NewFeatureCubit>().fetchNewFeatureData(
                  "TestValue",
                  123,
                );
              },
              child: Text("Fetch Data"),
            ),
          );
        },
      ),
    );
  }
}
```
✅ The UI now allows users to make API calls and see the results.
## 🔁 Complete Flow Recap
1. User clicks a button in the UI

2. UI triggers the Bloc (NewFeatureCubit)

3. Bloc calls DataManager to make the API request

4. DataManager calls NetworkApiClient (Retrofit)

5. Retrofit sends the request and receives a response

6. Response is returned to the UI via Bloc

7. UI updates based on API response

## ✅ Now, you have a structured way to:

1. Define new API endpoints

2. Manage request and response models

3. Handle API calls efficiently using Bloc & DataManager

4. Display data dynamically in the UI

## 🛠 Step 7: Update the DataManagerImp Interface
To maintain consistency, update ```dataManagerImp.dart```:
```dart
import '../dataProvider/model/requestBody/new_feature_request_body.dart';
import '../dataProvider/model/response/new_feature_response.dart';

abstract class DataManagerImp {
  Future<MainResponseModel> signIn(SignInBody signInBody);
  Future<MainResponseModel> signUp(SignUpBody signUpBody);
  Future<MainResponseModel> getCards();
  Future<dynamic> deleteCard(int cardId);
  Future<MainResponseModel> addCard(AddCardBody addCardBody);
  Future<MainResponseModel> createStripeToken(StripeTokenBody stripTokenBody);
  Future<MainResponseModel> getReceipt(ReceiptBody receiptBody);
  Future<MainResponseModel> bookRides(BookRidesBody bookRidesBody);

  // ✅ Add the new method
  Future<NewFeatureResponse> newFeature(NewFeatureRequestBody body);
}
```

### ✅ Why update the interface?

1. Ensures consistency → Any class implementing ```DataManagerImp``` must define ```newFeature()```.

2. Improves maintainability → Clearly defines all expected methods.

3. Better abstraction → Other parts of the code can rely on ```DataManagerImp``` rather than ```DataManager```.

<p></p>









## Second Method

To create a Retrofit request body for this JSON in Dart (Flutter), follow these steps:

## 📌 Step 1: Create the Request Body Model
Use @JsonSerializable() to make it easy to parse from/to JSON.

## 1️⃣ Install json_serializable and build_runner
Add these dependencies in your ```pubspec.yaml```:

```yaml
dependencies:
  json_annotation: ^4.8.1

dev_dependencies:
  build_runner: ^2.4.8
  json_serializable: ^6.7.1
```
Then, run:

```sh
flutter pub get
```
## 2️⃣ Define the Request Body Model (BookJourneyBody.dart)
```dart
import 'package:json_annotation/json_annotation.dart';

part 'book_journey_body.g.dart';

@JsonSerializable()
class BookJourneyBody {
  final String journeyId;
  final String startLocation;
  final String endLocation;
  final String startTime;
  final String endTime;
  final List<Passenger> passengers;
  final String cardId;
  final int totalAmount;
  final bool testMode;

  BookJourneyBody({
    required this.journeyId,
    required this.startLocation,
    required this.endLocation,
    required this.startTime,
    required this.endTime,
    required this.passengers,
    required this.cardId,
    required this.totalAmount,
    required this.testMode,
  });

  factory BookJourneyBody.fromJson(Map<String, dynamic> json) => _$BookJourneyBodyFromJson(json);
  Map<String, dynamic> toJson() => _$BookJourneyBodyToJson(this);
}

@JsonSerializable()
class Passenger {
  final String name;
  final String email;
  final String type;

  Passenger({required this.name, required this.email, required this.type});

  factory Passenger.fromJson(Map<String, dynamic> json) => _$PassengerFromJson(json);
  Map<String, dynamic> toJson() => _$PassengerToJson(this);
}
```
## 3️⃣ Generate the JSON Serialization Code
Run this command:

```sh
flutter pub run build_runner build --delete-conflicting-outputs
```
This will generate a book_journey_body.g.dart file.

## 📌 Step 2: Define the Retrofit API Interface
Create or update your network_api_client.dart:

```dart
import 'package:dio/dio.dart';
import 'package:retrofit/retrofit.dart';
import '../model/requestBody/book_journey_body.dart';

part 'network_api_client.g.dart';

@RestApi(baseUrl: "http://18.116.82.145:80/api/v1/")
abstract class NetworkApiClient {
  factory NetworkApiClient(Dio dio, {String baseUrl}) = _NetworkApiClient;

  @POST("user/book-journey")
  Future<Response> bookJourney(@Body() BookJourneyBody bookJourneyBody);
}
```
## 📌 Step 3: Create and Use the API Call
In your repository or provider, use this function:

```dart
Future<void> bookJourneyRequest() async {
  final dio = Dio();
  final apiClient = NetworkApiClient(dio);

  try {
    final requestBody = BookJourneyBody(
      journeyId: "journey_test_12345",
      startLocation: "London Test Station",
      endLocation: "Manchester Piccadilly",
      startTime: "2023-12-15T08:30:00Z",
      endTime: "2023-12-15T10:15:00Z",
      passengers: [
        Passenger(name: "John Doe", email: "john@example.com", type: "adult"),
      ],
      cardId: "card_12345",
      totalAmount: 4500,
      testMode: true,
    );

    final response = await apiClient.bookJourney(requestBody);
    print("Booking Success: ${response.data}");
  } catch (e) {
    print("Error booking journey: $e");
  }
}
```
### 🎯 Summary
1. ✔ Created a request body model (BookJourneyBody.dart)
2. ✔ Used json_serializable for easy JSON conversion
3. ✔ Defined Retrofit API call in NetworkApiClient.dart
4. ✔ Created and tested the API request in bookJourneyRequest()

Now your Flutter app can send a booking request successfully! 🚀


## 📌 Making a GET Request Using Retrofit in Flutter (Dio)
If you're making a GET request that doesn't have a body, you'll follow these steps:

## 1️⃣ Add Dependencies
Ensure you have dio and retrofit in your ```pubspec.yaml```:
```yaml
dependencies:
  dio: ^5.4.1
  retrofit: ^4.1.0
  json_annotation: ^4.8.1

dev_dependencies:
  retrofit_generator: ^7.0.8
  build_runner: ^2.4.6
```
Then run:
```sh
flutter pub get
```
## 2️⃣ Create the Response Model
Since the API returns JSON, we need to map it to a Dart model.
Let's say our API returns a list of users, like this:
```json
{
  "status": "success",
  "message": "Users retrieved successfully",
  "data": [
    {
      "id": 1,
      "name": "John Doe",
      "email": "john.doe@example.com"
    },
    {
      "id": 2,
      "name": "Jane Smith",
      "email": "jane.smith@example.com"
    }
  ]
}
```
Create ```UserResponse.dart```
```dart
import 'package:freezed_annotation/freezed_annotation.dart';

part 'user_response.g.dart';
part 'user_response.freezed.dart';

@freezed
class UserResponse with _$UserResponse {
  const factory UserResponse({
    required String status,
    required String message,
    required List<User> data,
  }) = _UserResponse;

  factory UserResponse.fromJson(Map<String, dynamic> json) =>
      _$UserResponseFromJson(json);
}

@freezed
class User with _$User {
  const factory User({
    required int id,
    required String name,
    required String email,
  }) = _User;

  factory User.fromJson(Map<String, dynamic> json) => _$UserFromJson(json);
}
```
Then run:
```sh
flutter pub run build_runner build --delete-conflicting-outputs
```
## 3️⃣ Define the Retrofit API Client
Now, create the API interface for Retrofit.

Create ```NetworkApiClient.dart```
```dart
import 'package:dio/dio.dart';
import 'package:retrofit/retrofit.dart';
import '../model/response/user_response.dart';

part 'network_api_client.g.dart';

@RestApi(baseUrl: "https://api.example.com/v1/")
abstract class NetworkApiClient {
  factory NetworkApiClient(Dio dio, {String baseUrl}) = _NetworkApiClient;

  @GET("users") // 👈 This is a GET request without a body
  Future<UserResponse> getUsers();
}
```
Then run:
```sh
flutter pub run build_runner build --delete-conflicting-outputs
```
## 4️⃣ Implement the API Call in DataManager
We use DataManager to handle network requests in a structured way.

Modify ```DataManager.dart```
```dart
import '../dataProvider/model/response/user_response.dart';
import '../network/network_api_client.dart';

class DataManager {
  final NetworkApiClient _apiClient;

  DataManager({required NetworkApiClient apiClient}) : _apiClient = apiClient;

  Future<UserResponse> fetchUsers() async {
    try {
      return await _apiClient.getUsers();
    } catch (e) {
      throw Exception("Failed to fetch users: $e");
    }
  }
}
```
## 5️⃣ Create a Bloc for State Management (Optional)
To manage state efficiently, let's use Cubit.

Create ```user_cubit.dart```
```dart
import 'package:flutter_bloc/flutter_bloc.dart';
import '../../dataProvider/model/response/user_response.dart';
import '../../dataManager/data_manager.dart';

abstract class UserState {}

class UserInitial extends UserState {}
class UserLoading extends UserState {}
class UserLoaded extends UserState {
  final UserResponse response;
  UserLoaded(this.response);
}
class UserError extends UserState {
  final String message;
  UserError(this.message);
}

class UserCubit extends Cubit<UserState> {
  final DataManager _dataManager;

  UserCubit(this._dataManager) : super(UserInitial());

  Future<void> loadUsers() async {
    emit(UserLoading());
    try {
      final response = await _dataManager.fetchUsers();
      emit(UserLoaded(response));
    } catch (e) {
      emit(UserError("Failed to load users"));
    }
  }
}
```
## 6️⃣ Call the API from the UI
Finally, let's call the API and display the data.

Modify ```UserScreen.dart```
```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import '../bloc/user_cubit.dart';

class UserScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Users List")),
      body: BlocConsumer<UserCubit, UserState>(
        listener: (context, state) {
          if (state is UserError) {
            ScaffoldMessenger.of(context).showSnackBar(
              SnackBar(content: Text(state.message)),
            );
          }
        },
        builder: (context, state) {
          if (state is UserLoading) {
            return Center(child: CircularProgressIndicator());
          } else if (state is UserLoaded) {
            return ListView.builder(
              itemCount: state.response.data.length,
              itemBuilder: (context, index) {
                final user = state.response.data[index];
                return ListTile(
                  title: Text(user.name),
                  subtitle: Text(user.email),
                );
              },
            );
          }
          return Center(
            child: ElevatedButton(
              onPressed: () {
                context.read<UserCubit>().loadUsers();
              },
              child: Text("Load Users"),
            ),
          );
        },
      ),
    );
  }
}
```
## 7️⃣ Run and Test
Start your app and navigate to UserScreen. Press the Load Users button, and you should see the list appear. 🚀
## 🔁 Full Flow Recap
1. User taps "Load Users" button
2.  UI triggers ```UserCubit.loadUsers()```
3.  Cubit calls ```DataManager.fetchUsers()```
4.  DataManager calls ```NetworkApiClient.getUsers()```
5.  Retrofit (Dio) sends a ```GET``` request
6.  API responds with user data
7.  Cubit updates the UI with ```UserLoaded``` state
8.  UI displays the list of users









