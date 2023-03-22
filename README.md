# Advance-android-examples
Some advance topic examples for kotlin android

# Advanced MVVM Example with Kotlin and ViewBinding

```
This example demonstrates how to create an Android app using the Model-View-ViewModel (MVVM) pattern with Kotlin, Android Architecture Components, ViewBinding, and Retrofit. The app displays a list of repositories fetched from GitHub's API.

```

## 1. Add dependencies in build.gradle (Module):

```groovy
dependencies {
    implementation 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.4.1'
    implementation 'androidx.lifecycle:lifecycle-livedata-ktx:2.4.1'
    implementation 'androidx.lifecycle:lifecycle-runtime-ktx:2.4.1'
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
}
```

## 2. Enable ViewBinding in build.gradle (Module):

```
android {
    ...
    buildFeatures {
        viewBinding true
    }
    ...
}
```

## 3. Create a data class for the Repository:

```
data class Repository(
    val id: Long,
    val name: String,
    val description: String?,
    val stargazers_count: Int,
    val forks_count: Int,
    val language: String?
)
```

## 4. Define the API interface:

```
import retrofit2.Call
import retrofit2.http.GET

interface GitHubApi {
    @GET("users/{user}/repos")
    fun getRepositories(): Call<List<Repository>>
}
```

## 5. Create a Retrofit instance:

```
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

object RetrofitClient {
    private const val BASE_URL = "https://api.github.com/"

    val instance: GitHubApi by lazy {
        val retrofit = Retrofit.Builder()
            .baseUrl(BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .build()

        retrofit.create(GitHubApi::class.java)
    }
}
```

## 6. Create a Repository ViewModel:

```
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.launch
import retrofit2.Response

class RepoViewModel : ViewModel() {
    val repositories: MutableLiveData<Response<List<Repository>>> = MutableLiveData()

    fun getRepositories() {
        viewModelScope.launch(Dispatchers.IO) {
            val response = RetrofitClient.instance.getRepositories()
            repositories.postValue(response)
        }
    }
}
```

## 7. Implement the RecyclerView adapter with ViewBinding:

```
import android.view.LayoutInflater
import android.view.ViewGroup
import androidx.recyclerview.widget.RecyclerView
import com.example.myapplication.databinding.RepoItemBinding

class RepoAdapter(private val repoList: List<Repository>) : RecyclerView.Adapter<RepoAdapter.ViewHolder>() {

    class ViewHolder(val binding: RepoItemBinding) : RecyclerView.ViewHolder(binding.root)

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
        val binding = RepoItemBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        return ViewHolder(binding)
    }

    override fun onBindViewHolder(holder: ViewHolder, position: Int) {
        val repo = repoList[position]
        with(holder.binding) {
            repoName.text = repo.name
            repoDescription.text = repo.description ?: "No description"
            repoStars.text = repo.stargazers_count.toString()
            repoForks.text = repo.forks_count.toString()
            repoLanguage.text = repo.language ?: "No language"
        }
    }
    
    override fun getItemCount() = repoList.size
}
```

## 8. Implement the MainActivity with ViewBinding:

```
import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import androidx.activity.viewModels
import androidx.lifecycle.Observer
import androidx.recyclerview.widget.LinearLayoutManager
import android.widget.Toast
import com.example.myapplication.databinding.ActivityMainBinding

class MainActivity : AppCompatActivity() {
    private val repoViewModel: RepoViewModel by viewModels()
    private lateinit var repoAdapter: RepoAdapter
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        binding.recyclerView.layoutManager = LinearLayoutManager(this)

        repoViewModel.getRepositories()

        repoViewModel.repositories.observe(this, Observer { response ->
            if (response.isSuccessful && response.body() != null) {
                repoAdapter = RepoAdapter(response.body()!!)
                binding.recyclerView.adapter = repoAdapter
            } else {
                Toast.makeText(this, "Failed to load repositories", Toast.LENGTH_SHORT).show()
            }
        })
    }
}
```

## 9. Add necessary XML layout files:

activity_main.xml:

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recycler_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
</androidx.constraintlayout.widget.ConstraintLayout>
```

repo_item.xml:

```
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="16dp">

    <TextView
        android:id="@+id/repo_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="18sp"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/repo_description"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_constraintStart_toStartOf="@+id/repo_name"
        app:layout_constraintTop_toBottomOf="@+id/repo_name" />

    <TextView
        android:id="@+id/repo_stars"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_constraintStart_toStartOf="@+id/repo_description"
        app:layout_constraintTop_toBottomOf="@+id/repo_description" />

    <TextView
        android:id="@+id/repo_forks"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_constraintStart_toEndOf
```
