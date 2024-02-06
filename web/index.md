#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#pragma pack(1)

typedef struct
{
    char signature[2];
    unsigned int fileSize;
    short reserved1;
    short reserved2;
    unsigned int dataOffset;
} BMPHeader;

typedef struct
{
    unsigned int size;
    int width;
    int height;
    short planes;
    short bitCount;
    unsigned int compression;
    unsigned int imageSize;
    int xPixelsPerMeter;
    int yPixelsPerMeter;
    unsigned int colorsUsed;
    unsigned int colorsImportant;
} BMPInfoHeader;

void convertToGrayscale(const char *inputFileName, const char *outputFileName);

int main(int argc, char *argv[])
{
    if (argc != 3)
    {
        printf("Usage: %s input_file output_file\n", argv[0]);
        return 1;
    }

    const char *inputFileName = argv[1];
    const char *outputFileName = argv[2];

    convertToGrayscale(inputFileName, outputFileName);

    printf("Conversion completed.\n");

    return 0;
}

void convertToGrayscale(const char *inputFileName, const char *outputFileName)
{
    FILE *inputFile = fopen(inputFileName, "rb");

    if (inputFile == NULL)
    {
        perror("Error opening input file");
        exit(EXIT_FAILURE);
    }

    fseek(inputFile, 0, SEEK_END);
    long fileSize = ftell(inputFile);
    rewind(inputFile);

    BMPHeader bmpHeader;
    BMPInfoHeader bmpInfoHeader;

    fread(&bmpHeader, sizeof(BMPHeader), 1, inputFile);
    fread(&bmpInfoHeader, sizeof(BMPInfoHeader), 1, inputFile);

    if (bmpHeader.signature[0] != 'B' || bmpHeader.signature[1] != 'M')
    {
        fprintf(stderr, "Error: Input file is not a valid BMP file\n");
        fclose(inputFile);
        exit(EXIT_FAILURE);
    }

    int width = bmpInfoHeader.width;
    int height = bmpInfoHeader.height;

    // Calculate row padding
    int padding = (4 - (width * sizeof(unsigned char) % 4)) % 4;

    // Calculate image data size
    int imageDataSize = width * height * sizeof(unsigned char);

    // Allocate memory for image data
    unsigned char *imageData = malloc(imageDataSize);
    if (imageData == NULL)
    {
        perror("Error allocating memory for image data");
        fclose(inputFile);
        exit(EXIT_FAILURE);
    }
    fread(imageData, sizeof(unsigned char), imageDataSize, inputFile);

    fclose(inputFile);

    // Open the output file for writing
    FILE *outputFile = fopen(outputFileName, "wb");

    if (outputFile == NULL)
    {
        perror("Error opening output file");
        free(imageData);
        exit(EXIT_FAILURE);
    }

    // Write the headers to the output file
    fwrite(&bmpHeader, sizeof(BMPHeader), 1, outputFile);
    fwrite(&bmpInfoHeader, sizeof(BMPInfoHeader), 1, outputFile);

    // Convert image to grayscale
    for (int i = 0; i < imageDataSize; i += 3)
    {
        // Grayscale value using weighted average formula
        unsigned char grayValue = (0.2126 * imageData[i] + 0.7152 * imageData[i + 1] + 0.0722 * imageData[i + 2]);

        // Write the grayscale value to all three color channels
        fwrite(&grayValue, sizeof(unsigned char), 1, outputFile);
        fwrite(&grayValue, sizeof(unsigned char), 1, outputFile);
        fwrite(&grayValue, sizeof(unsigned char), 1, outputFile);

        // Write padding to the output file
        if (padding > 0)
        {
            for (int j = 0; j < padding; j++)
            {
                fwrite(&grayValue, sizeof(unsigned char), 1, outputFile);
            }
        }
    }

    // Close the output file
    fclose(outputFile);

    free(imageData);
}
