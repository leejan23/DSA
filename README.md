#include <iostream>
#include <vector>
#include <algorithm>
#include <ctime>
#include <cstdlib>
#include <limits>
#include <chrono>

using namespace std;

vector<int> generateRandomNumbers(int n) {
    vector<int> nums(n);
    static bool seeded = false;
    if (!seeded) {
        srand(time(0));
        seeded = true;
    }
    for (int i = 0; i < n; ++i)
        nums[i] = rand() % (n * 10) + 1;
    return nums;
}

void printVector(const vector<int>& arr) {
    if (arr.empty()) {
        cout << "[Empty]\n";
        return;
    }
    for (int x : arr)
        cout << x << " ";
    cout << endl;
}

int getValidatedIntegerInput() {
    int num;
    while (!(cin >> num)) {
        cout << "Invalid input. Please enter a number: ";
        cin.clear();
        cin.ignore(numeric_limits<streamsize>::max(), '\n');
    }
    cin.ignore(numeric_limits<streamsize>::max(), '\n');
    return num;
}

int binarySearch(const vector<int>& arr, int target) {
    int low = 0, high = arr.size() - 1;
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (arr[mid] == target) return mid;
        (arr[mid] < target) ? low = mid + 1 : high = mid - 1;
    }
    return -1;
}

int binarySearchForExp(const vector<int>& arr, int low, int high, int target) {
    while (low <= high) {
        int mid = low + (high - low) / 2;
        if (arr[mid] == target) return mid;
        (arr[mid] < target) ? low = mid + 1 : high = mid - 1;
    }
    return -1;
}

int exponentialSearch(const vector<int>& arr, int target) {
    int n = arr.size();
    if (n == 0) return -1;
    if (arr[0] == target) return 0;
    int i = 1;
    while (i < n && arr[i] <= target) i *= 2;
    return binarySearchForExp(arr, i / 2, min(i, n - 1), target);
}

int partition(vector<int>& arr, int low, int high) {
    int pivot = arr[high], i = low - 1;
    for (int j = low; j < high; j++) {
        if (arr[j] <= pivot) {
            i++;
            swap(arr[i], arr[j]);
        }
    }
    swap(arr[i + 1], arr[high]);
    return i + 1;
}

void quickSort(vector<int>& arr, int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}

void merge(vector<int>& arr, int l, int m, int r) {
    int n1 = m - l + 1, n2 = r - m;
    vector<int> L(n1), R(n2);
    for (int i = 0; i < n1; i++) L[i] = arr[l + i];
    for (int j = 0; j < n2; j++) R[j] = arr[m + 1 + j];

    int i = 0, j = 0, k = l;
    while (i < n1 && j < n2)
        arr[k++] = (L[i] <= R[j]) ? L[i++] : R[j++];
    while (i < n1) arr[k++] = L[i++];
    while (j < n2) arr[k++] = R[j++];
}

void mergeSort(vector<int>& arr, int l, int r) {
    if (l < r) {
        int m = l + (r - l) / 2;
        mergeSort(arr, l, m);
        mergeSort(arr, m + 1, r);
        merge(arr, l, m, r);
    }
}

int main() {
    int main_choice, algo_choice, n;
    vector<int> data, temp;

    chrono::time_point<chrono::high_resolution_clock> start, end;
    chrono::nanoseconds duration;

    do {
        cout << "\n--- Console App ---\n";
        cout << "1. Sort\n2. Search\n0. Exit\nEnter choice: ";
        main_choice = getValidatedIntegerInput();

        if (main_choice == 1 || main_choice == 2) {
            cout << "Enter number of elements (N): ";
            n = getValidatedIntegerInput();
            if (n <= 0) {
                cout << "N must be positive.\n";
                continue;
            }
            data = generateRandomNumbers(n);
            cout << "Original array: ";
            printVector(data);
        }

        switch (main_choice) {
            case 1: {
                cout << "\nSort Algorithm:\n1. Quick Sort\n2. Merge Sort\nChoice: ";
                algo_choice = getValidatedIntegerInput();
                temp = data;
                start = chrono::high_resolution_clock::now();
                if (algo_choice == 1)
                    quickSort(temp, 0, temp.size() - 1);
                else if (algo_choice == 2)
                    mergeSort(temp, 0, temp.size() - 1);
                else {
                    cout << "Invalid sorting choice.\n";
                    break;
                }
                end = chrono::high_resolution_clock::now();
                duration = chrono::duration_cast<chrono::nanoseconds>(end - start);
                cout << "Time taken: " << duration.count() / 1000.0 << " µs ("
                     << duration.count() << " ns)\n";
                cout << "Sorted array: ";
                printVector(temp);
                break;
            }

            case 2: {
                sort(data.begin(), data.end());
                cout << "Sorted for search: ";
                printVector(data);
                cout << "Enter target: ";
                int target = getValidatedIntegerInput();
                cout << "\nSearch Algorithm:\n1. Binary\n2. Exponential\nChoice: ";
                algo_choice = getValidatedIntegerInput();

                int result = -1;
                const int reps = 100000;
                start = chrono::high_resolution_clock::now();
                for (int i = 0; i < reps; ++i) {
                    result = (algo_choice == 1)
                             ? binarySearch(data, target)
                             : exponentialSearch(data, target);
                }
                end = chrono::high_resolution_clock::now();
                duration = chrono::duration_cast<chrono::nanoseconds>(end - start);
                double avg = static_cast<double>(duration.count()) / reps;
                cout << "Avg time/search: " << avg / 1000.0 << " µs (" << avg << " ns)\n";
                if (result != -1)
                    cout << "Target " << target << " found at index: " << result << "\n";
                else
                    cout << "Target " << target << " not found.\n";
                break;
            }

            case 0:
                cout << "Goodbye!\n";
                break;

            default:
                cout << "Invalid choice.\n";
                break;
        }

    } while (main_choice != 0);

    return 0;
}
