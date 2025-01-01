% Fruit Rot Detection Script - Final Corrected Version

% Clear workspace and command window
clc;
clear;
close all;

% Prompt user to select an image
[file, path] = uigetfile('C:\Users\21b-042-ce\Downloads\img6.jfif');
if isequal(file, 0)
    disp('No file selected. Exiting...');
    return;
end

% Read and resize the selected image
img = imread(fullfile(path, file));
img = imresize(img, [500, 500]);

% Display the original image
figure;
imshow(img);
title('Original Image');

%% Convert Image to Different Color Spaces
hsvImg = rgb2hsv(img); % HSV color space
grayImg = rgb2gray(img); % Grayscale image
rChannel = img(:, :, 1); % Red channel for detecting reddish-brown areas

%% Automatic Thresholding for Dark Areas (Otsu's Method)
% Otsu's method automatically calculates the threshold for dark spots
darkThreshold = graythresh(grayImg); 
darkMask = grayImg < darkThreshold * 255; % Thresholded dark spots mask

%% Detect Brownish Hue Regions in HSV
% Hue range for brown: ~0.05 to ~0.15 (can vary slightly)
hueMask = (hsvImg(:, :, 1) >= 0.05) & (hsvImg(:, :, 1) <= 0.15);

% Combine hue mask with low saturation to ensure "brown rot" detection
lowSaturationMask = hsvImg(:, :, 2) < 0.4; % Less vibrant regions
brownRotMask = hueMask & lowSaturationMask;

%% Detect Low Brightness Areas
% Automatically calculate threshold for low brightness
brightnessThreshold = graythresh(hsvImg(:, :, 3));
lowBrightnessMask = hsvImg(:, :, 3) < brightnessThreshold;

%% Combine All Masks
% Rotten areas are defined by:
% - Dark spots (low grayscale values)
% - Brownish regions (hue and low saturation)
% - Low brightness areas
combinedMask = darkMask | brownRotMask | lowBrightnessMask;

%% Morphological Operations for Mask Refinement
combinedMask = imopen(combinedMask, strel('disk', 5)); % Remove small noise
combinedMask = imclose(combinedMask, strel('disk', 10)); % Fill gaps
combinedMask = imfill(combinedMask, 'holes'); % Fill any internal holes

%% Display the Rotten Areas Mask
figure;
imshow(combinedMask);
title('Detected Rotten Areas Mask');

%% Calculate the Percentage of Rotten Areas
totalPixels = numel(combinedMask);
rottenPixels = sum(combinedMask(:));
rottenPercentage = (rottenPixels / totalPixels) * 100;

% Print the result
fprintf('The fruit is %.2f%% rotten.\n', rottenPercentage);

%% Overlay Rotten Areas on the Original Image
% Highlight detected areas in red
overlayImg = img;
overlayImg(repmat(combinedMask, [1, 1, 3])) = 255; % Turn rotten areas white for visualization
redOverlay = cat(3, uint8(combinedMask) * 255, zeros(size(combinedMask), 'uint8'), zeros(size(combinedMask), 'uint8'));
finalOverlay = imadd(overlayImg, redOverlay);

% Display the final result
figure;
imshow(finalOverlay);
title(sprintf('Rotten Areas Highlighted (%.2f%% Rotten)', rottenPercentage));
