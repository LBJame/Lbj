#include <iostream>
#include <opencv2/core.hpp>
#include <opencv2/highgui.hpp>
#include<opencv2/opencv.hpp>
#include <math.h>
using namespace std;
using namespace cv;


int main()
{
	Mat I1 = imread("1.bmp");
	cvtColor(I1, I1, CV_BGR2GRAY);
	int m = getOptimalDFTSize(I1.rows);
	int n = getOptimalDFTSize(I1.cols);
	Mat padded;
	Mat real, ima;
	copyMakeBorder(I1, padded, 0, m - I1.rows, 0, n - I1.cols, BORDER_CONSTANT, Scalar::all(0));
	for(int i=0;i<=padded.rows;i++)
		for (int j = 0; j <= padded.cols; j++)
		{
			ima.at<uchar>(i, j) = sin(padded.at<uchar>(i, j));
			real.at<uchar>(i, j) = cos(padded.at<uchar>(i, j));
		}
	Mat planes[] = { real,ima };
	Mat complexI;
	merge(planes, 2, complexI);
	//Mat planes[] = { Mat_<float>(padded),Mat::zeros(padded.size(),CV_32F) };
	//Mat complexI;
	//merge(planes, 2, complexI);
	dft(complexI, complexI);
	split(complexI, planes);
	magnitude(planes[0], planes[1], planes[0]);
	Mat magnitudeImage = planes[0];
	magnitudeImage += Scalar::all(1);
	log(magnitudeImage, magnitudeImage);
	magnitudeImage = magnitudeImage(Rect(0, 0, magnitudeImage.cols &-2, magnitudeImage.rows &-2));
	int cx = magnitudeImage.cols / 2;
	int cy = magnitudeImage.rows / 2;
	Mat q0(magnitudeImage, Rect(0, 0, cx, cy));
	Mat q1(magnitudeImage, Rect(cx, 0, cx, cy));
	Mat q2(magnitudeImage, Rect(0, cy, cx, cy));
	Mat q3(magnitudeImage, Rect(cx, cy, cx, cy));
	Mat tmp;
	q0.copyTo(tmp);
	q3.copyTo(q0);
	tmp.copyTo(q3);
	q1.copyTo(tmp);
	q2.copyTo(q1);
	tmp.copyTo(q2);
	normalize(magnitudeImage, magnitudeImage, 0, 1, NORM_MINMAX);
	imshow("Fourier", magnitudeImage);
	waitKey();
	
	
	return 0;
	
}
