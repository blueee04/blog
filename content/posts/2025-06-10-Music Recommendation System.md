# 🎵 Music Recommendation System: Project Overview

Welcome to my project on building a content-based and hybrid music recommendation system! This post covers the motivation, dataset, methodology, results, and key takeaways from my work.

## 📌 Table of Contents

1. [Introduction](#introduction)
2. [Dataset](#dataset)
3. [Feature Extraction](#feature-extraction)
4. [Model Architecture](#model-architecture)
5. [Evaluation & Results](#evaluation--results)
6. [Screenshots & Visualizations](#screenshots--visualizations)
7. [Conclusion & Future Work](#conclusion--future-work)

## 📝 Introduction

Music recommendation systems help users discover new songs tailored to their tastes. In this project, I implemented a system that leverages audio features and machine learning to recommend songs based on content and user preferences.

## 🎼 Dataset

- **Source:** GTZAN Genre Collection
- **Genres:** Blues, Classical, Country, Disco, HipHop, Jazz, Metal, Pop, Reggae, Rock
- **Files:** 100 audio files per genre, 10 genres
- **Format:** .wav audio files, 30 seconds each

**Directory Structure:**
```
music_recommendation_system_project/
  Data/
    genres_original/
      blues/
      classical/
      ...
    images_original/
      blues/
      classical/
      ...
```

![Dataset Structure](https://raw.githubusercontent.com/blueee04/blog/main/content/images/2025-06-10-Music%20Recommendation%20System/genre_distribution.png)
*Genre distribution in the dataset.*

## 🛠️ Feature Extraction

Used librosa to extract:
- MFCCs (Mel-Frequency Cepstral Coefficients)
- Chroma features
- Spectral features (centroid, bandwidth, rolloff)
- Zero-crossing rate, tempo, etc.
- Features saved in CSV files for 3-sec and 30-sec segments.

```python
# Example: Extracting MFCCs
mfccs = librosa.feature.mfcc(y=y, sr=sr, n_mfcc=20)
```

## 🧠 Model Architecture

- **Preprocessing:** Standardization, PCA for dimensionality reduction
- **Genre Classification:** Random Forest Classifier
- **Recommendation Methods:**
  - Content-based (cosine similarity, DTW on MFCCs)
  - Genre-based (simulated collaborative filtering)
  - Hybrid (weighted combination)

> ![Model Pipeline](https://raw.githubusercontent.com/blueee04/blog/main/content/images/2025-06-10-Music%20Recommendation%20System/pipeline.png)
*Model architecture and processing pipeline.*

## 📊 Evaluation & Results

- **Metrics:** Accuracy, confusion matrix, classification report
- **Visualization:** Confusion matrix, feature importance, genre distribution

> ![Confusion Matrix](https://raw.githubusercontent.com/blueee04/blog/main/content/images/2025-06-10-Music%20Recommendation%20System/confusion_matrix.png)
*Confusion matrix for genre classification results.*

```text
Sample Recommendations:
1. song1.wav (Rock) - Score: 0.92
2. song2.wav (Blues) - Score: 0.89
3. song3.wav (Metal) - Score: 0.87
...
```


> ![Feature Importance](https://raw.githubusercontent.com/blueee04/blog/main/content/images/2025-06-10-Music%20Recommendation%20System/feature_importance.png)
*Feature importance plot for the Random Forest classifier.*

## 🖥️ App Screenshot

Here is a screenshot of the music recommendation app interface:

![App Main Interface](https://raw.githubusercontent.com/blueee04/blog/main/content/images/2025-06-10-Music%20Recommendation%20System/app.png)
*Main interface of the music recommendation app.*

## 🚀 Conclusion & Future Work

### Achievements:
- Built a working music recommendation system using audio features
- Achieved high accuracy in genre classification
- Generated meaningful song recommendations

### Next Steps:
- Integrate user feedback for collaborative filtering
- Deploy as a web app for real-time recommendations
- Explore deep learning approaches for feature extraction

## 📂 Resources

- [GitHub Repository](https://github.com/yourusername/music-recommendation-system)
- [Librosa Documentation](https://librosa.org/doc/latest/)
- [PaperMod Theme](https://github.com/adityatelange/hugo-PaperMod)

---

*Feel free to reach out for collaboration or questions!*