#include <stdio.h>
#include <stdlib.h>

#define ARRAY_SIZE 10

static void swap_elements(int *left, int *right)
{
    int temporary = *left;
    *left = *right;
    *right = temporary;
}

static size_t partition_array(int array[], size_t low, size_t high)
{
    int pivot = array[high];
    size_t boundary = low;

    for (size_t index = low; index < high; ++index) {
        if (array[index] < pivot) {
            swap_elements(&array[boundary], &array[index]);
            ++boundary;
        }
    }

    swap_elements(&array[boundary], &array[high]);
    return boundary;
}

static void quick_sort_range(int array[], size_t low, size_t high)
{
    if (low >= high) {
        return;
    }

    size_t pivot_index = partition_array(array, low, high);

    if (pivot_index > low) {
        quick_sort_range(array, low, pivot_index - 1);
    }
    if (pivot_index < high) {
        quick_sort_range(array, pivot_index + 1, high);
    }
}

static int quick_sort(int array[], size_t size)
{
    if (array == NULL && size > 0) {
        return -1;
    }
    if (size > 1) {
        quick_sort_range(array, 0, size - 1);
    }
    return 0;
}

static void print_array(const int array[], size_t size)
{
    for (size_t index = 0; index < size; ++index) {
        printf("%d%s", array[index], index + 1 == size ? "" : " ");
    }
    putchar('\n');
}

int main(void)
{
    const int initial_values[ARRAY_SIZE] = {10, 7, 8, 9, 1, 5, 2, 3, 4, 6};
    int *data = malloc(ARRAY_SIZE * sizeof *data);
    if (data == NULL) {
        fputs("Error: unable to allocate memory.\n", stderr);
        return EXIT_FAILURE;
    }

    for (size_t index = 0; index < ARRAY_SIZE; ++index) {
        data[index] = initial_values[index];
    }

    puts("Original array:");
    print_array(data, ARRAY_SIZE);

    if (quick_sort(data, ARRAY_SIZE) != 0) {
        fputs("Error: invalid array passed to quick_sort.\n", stderr);
        free(data);
        return EXIT_FAILURE;
    }

    puts("Sorted array:");
    print_array(data, ARRAY_SIZE);

    free(data);
    return EXIT_SUCCESS;
}
