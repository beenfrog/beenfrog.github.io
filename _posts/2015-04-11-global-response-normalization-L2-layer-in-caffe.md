---
layout: post
title: Global Response Normalization (L2) layer in caffe
tags:
- c/c++
- caffe
categories:
- code
comments: true
---

Sometimes we want to normalize the data in one layer, especially L2 Normalization.  However, there is not such layer in caffe, so I write the simple layer with the inspiration from the most similar layer called SoftmaxLayer. 

## 1. add protobuf
Add the new layer type GRN, which is short for Global Response Normalization
caffe/src/caffe/proto/caffe.proto

```lua
message LayerParameter {
//...
  // LayerType next available ID: 39 (last added: GRN)
  enum LayerType {
    //...
    GRN = 38;
    //...
    }
//...
}
```

## 2. add layer factory
caffe/src/caffe/layer_factory.cpp

```c++
template <typename Dtype>
Layer<Dtype>* GetLayer(const LayerParameter& param) {
//...
  switch (type) {
//...
  case LayerParameter_LayerType_GRN:
    return new GRNLayer<Dtype>(param);
  }
//...
}
```

## 3. class declaration
caffe/include/caffe/common_layers.hpp

```c++
/**
 * @brief Global Response Normalizion with L2 .
 *
 * TODO(dox): thorough documentation for Forward, Backward, and proto params.
 */
template <typename Dtype>
class GRNLayer : public Layer<Dtype> {
 public:
  explicit GRNLayer(const LayerParameter& param)
      : Layer<Dtype>(param) {}
  virtual void Reshape(const vector<Blob<Dtype>*>& bottom,
      vector<Blob<Dtype>*>* top);
  
  virtual inline LayerParameter_LayerType type() const {
    return LayerParameter_LayerType_GRN;
  }
  virtual inline int ExactNumBottomBlobs() const { return 1; }
  virtual inline int ExactNumTopBlobs() const { return 1; }

 protected:
  virtual void Forward_cpu(const vector<Blob<Dtype>*>& bottom,
      vector<Blob<Dtype>*>* top);
  virtual void Forward_gpu(const vector<Blob<Dtype>*>& bottom,
      vector<Blob<Dtype>*>* top);
  virtual void Backward_cpu(const vector<Blob<Dtype>*>& top,
      const vector<bool>& propagate_down, vector<Blob<Dtype>*>* bottom);
  virtual void Backward_gpu(const vector<Blob<Dtype>*>& top,
      const vector<bool>& propagate_down, vector<Blob<Dtype>*>* bottom);

  /// sum_multiplier is used to carry out sum using BLAS 1 x ch x 1 x 1
  Blob<Dtype> sum_multiplier_;
  /// square result n X ch x h x w
  Blob<Dtype> square_;
  /// norm is an intermediate Blob to hold temporary results. n * 1 * h * w
  Blob<Dtype> norm_;
  /// temp_dot n * 1 * h * w
  Blob<Dtype> temp_dot_;
};
```

## 4. cpp implementation
caffe/src/caffe/layers/grn_layer.cpp

```c++
#include <algorithm>
#include <vector>

#include "caffe/common_layers.hpp"
#include "caffe/layer.hpp"
#include "caffe/util/math_functions.hpp"

namespace caffe {

template <typename Dtype>
void GRNLayer<Dtype>::Reshape(const vector<Blob<Dtype>*>& bottom,
      vector<Blob<Dtype>*>* top) {
  (*top)[0]->Reshape(bottom[0]->num(), bottom[0]->channels(),
      bottom[0]->height(), bottom[0]->width());
  sum_multiplier_.Reshape(1, bottom[0]->channels(), 1, 1);
  Dtype* multiplier_data = sum_multiplier_.mutable_cpu_data();
  for (int i = 0; i < sum_multiplier_.count(); ++i) {
    multiplier_data[i] = 1.;
  }
  square_.Reshape(bottom[0]->num(), bottom[0]->channels(),
      bottom[0]->height(), bottom[0]->width());
  norm_.Reshape(bottom[0]->num(), 1, bottom[0]->height(), bottom[0]->width());
  temp_dot_.Reshape(bottom[0]->num(), 1, bottom[0]->height(), bottom[0]->width());
}

template <typename Dtype>
void GRNLayer<Dtype>::Forward_cpu(const vector<Blob<Dtype>*>& bottom,
    vector<Blob<Dtype>*>* top) {
  const Dtype* bottom_data = bottom[0]->cpu_data();
  Dtype* top_data = (*top)[0]->mutable_cpu_data();
  Dtype* square_data = square_.mutable_cpu_data();
  Dtype* norm_data = norm_.mutable_cpu_data();
  int num = bottom[0]->num();
  int channels = bottom[0]->channels();
  int dim = bottom[0]->count() / bottom[0]->num();
  int spatial_dim = bottom[0]->height() * bottom[0]->width();
  caffe_copy(bottom[0]->count(), bottom_data, top_data);
  caffe_copy(bottom[0]->count(), bottom_data, square_data);
  // do the normalizition.
  for (int i = 0; i < num; ++i) {
    // squre each element
    caffe_sqr<Dtype>(dim, square_data + i * dim, square_data + i * dim);
    // sum cross the channel
    caffe_cpu_gemv<Dtype>(CblasTrans, channels, spatial_dim, 1.,
        square_data + i * dim, sum_multiplier_.cpu_data(), 0.,
        norm_data + i * spatial_dim);
    // root the square norm_data
    caffe_powx<Dtype>(spatial_dim, norm_data + i * spatial_dim, 0.5,
        norm_data + i * spatial_dim);
    // division
    for (int j = 0; j < channels; j++) {
      caffe_div(spatial_dim, top_data + (*top)[0]->offset(i, j), 
          norm_data + i * spatial_dim, top_data + (*top)[0]->offset(i, j));
    }
  }
}

template <typename Dtype>
void GRNLayer<Dtype>::Backward_cpu(const vector<Blob<Dtype>*>& top,
    const vector<bool>& propagate_down,
    vector<Blob<Dtype>*>* bottom) {
  const Dtype* top_diff = top[0]->cpu_diff();
  const Dtype* top_data = top[0]->cpu_data();
  Dtype* bottom_diff = (*bottom)[0]->mutable_cpu_diff();
  const Dtype* bottom_data = (*bottom)[0]->cpu_data();
  Dtype* norm_data = norm_.mutable_cpu_data();
  Dtype* temp_dot_data = temp_dot_.mutable_cpu_data();
  Dtype* temp_data = square_.mutable_cpu_data();//just reuse the square_
  int num = top[0]->num();
  int channels = top[0]->channels();
  int dim = top[0]->count() / top[0]->num();
  int spatial_dim = top[0]->height() * top[0]->width();
  caffe_copy(top[0]->count(), top_diff, bottom_diff);
  for (int i = 0; i < num; ++i) {
    // b_diff = t_diff / norm - dot(t_diff, t_data) / (norm)^2 * bottom_data
    // compute dot(top_diff, top_data)
    for (int k = 0; k < spatial_dim; ++k) {
      temp_dot_data[i * spatial_dim + k] = caffe_cpu_strided_dot<Dtype>(channels,
          top_diff + i * dim + k, spatial_dim,
          top_data + i * dim + k, spatial_dim)  /  
          ( norm_data[i * spatial_dim + k] * norm_data[i * spatial_dim + k] );
    }
    // b_diff = t_diff / norm - dot(t_diff, t_data) / (norm)^2 * bottom_data
    for (int j = 0; j < channels; j++) {
      caffe_div(spatial_dim, bottom_diff + top[0]->offset(i, j), 
          norm_data + i * spatial_dim, bottom_diff + top[0]->offset(i, j));
      caffe_mul(spatial_dim, bottom_data + top[0]->offset(i, j), 
          temp_dot_data + i * spatial_dim, temp_data + top[0]->offset(i, j));
      caffe_axpy(spatial_dim, Dtype(-1.0), temp_data + top[0]->offset(i, j), 
          bottom_diff + top[0]->offset(i, j));
    }
  }
}


#ifdef CPU_ONLY
STUB_GPU(GRNLayer);
#endif

INSTANTIATE_CLASS(GRNLayer);


}  // namespace caffe

```

## 5. cuda implementation
caffe/src/caffe/layers/grn_layer.cu

```c++
#include <algorithm>
#include <vector>

#include "thrust/device_vector.h"

#include "caffe/common_layers.hpp"
#include "caffe/layer.hpp"
#include "caffe/util/math_functions.hpp"

namespace caffe {

template <typename Dtype>
__global__ void kernel_channel_sum(const int num, const int channels,
    const int spatial_dim, const Dtype* data, Dtype* channel_sum) {
  CUDA_KERNEL_LOOP(index, num * spatial_dim) {
    int n = index / spatial_dim;
    int s = index % spatial_dim;
    Dtype sum = 0;
    for (int c = 0; c < channels; ++c) {
      sum += data[(n * channels + c) * spatial_dim + s];
    }
    channel_sum[index] = sum;
  }
}

template <typename Dtype>
__global__ void kernel_channel_mul(const int num, const int channels,
    const int spatial_dim, Dtype* data, const Dtype* channel_sum) {
  CUDA_KERNEL_LOOP(index, num * spatial_dim) {
    int n = index / spatial_dim;
    int s = index % spatial_dim;
    for (int c = 0; c < channels; ++c) {
      data[(n * channels + c) * spatial_dim + s] *= channel_sum[index];
    }
  }
}

template <typename Dtype>
__global__ void kernel_channel_div(const int num, const int channels,
    const int spatial_dim, Dtype* data, const Dtype* channel_sum) {
  CUDA_KERNEL_LOOP(index, num * spatial_dim) {
    int n = index / spatial_dim;
    int s = index % spatial_dim;
    for (int c = 0; c < channels; ++c) {
      data[(n * channels + c) * spatial_dim + s] /= channel_sum[index];
    }
  }
}

template <typename Dtype>
__global__ void kernel_channel_dot(const int num, const int channels,
    const int spatial_dim, const Dtype* data_1, const Dtype* data_2,
    Dtype* channel_dot) {
  CUDA_KERNEL_LOOP(index, num * spatial_dim) {
    int n = index / spatial_dim;
    int s = index % spatial_dim;
    Dtype dot = 0;
    for (int c = 0; c < channels; ++c) {
      dot += (data_1[(n * channels + c) * spatial_dim + s]
          * data_2[(n * channels + c) * spatial_dim + s]);
    }
    channel_dot[index] = dot;
  }
}

template <typename Dtype>
void GRNLayer<Dtype>::Forward_gpu(const vector<Blob<Dtype>*>& bottom,
    vector<Blob<Dtype>*>* top) {
  const Dtype* bottom_data = bottom[0]->gpu_data();
  Dtype* top_data = (*top)[0]->mutable_gpu_data();
  Dtype* square_data = square_.mutable_gpu_data();
  Dtype* norm_data = norm_.mutable_gpu_data();
  int num = bottom[0]->num();
  int channels = bottom[0]->channels();
  int spatial_dim = bottom[0]->height() * bottom[0]->width();
  caffe_copy(bottom[0]->count(), bottom_data, top_data);
  caffe_copy(bottom[0]->count(), bottom_data, square_data);
  
  // square
  caffe_gpu_powx<Dtype>(bottom[0]->count(), square_data, Dtype(2.0), square_data);
  //sum cross channel
  kernel_channel_sum<Dtype><<<CAFFE_GET_BLOCKS(num * spatial_dim),
      CAFFE_CUDA_NUM_THREADS>>>(num, channels, spatial_dim, square_data,
      norm_data);
  // square root
  caffe_gpu_powx<Dtype>(num * spatial_dim, norm_data, Dtype(0.5), norm_data);
  // divide
  kernel_channel_div<Dtype><<<CAFFE_GET_BLOCKS(num * spatial_dim),
      CAFFE_CUDA_NUM_THREADS>>>(num, channels, spatial_dim, top_data,
      norm_data);
}

template <typename Dtype>
void GRNLayer<Dtype>::Backward_gpu(const vector<Blob<Dtype>*>& top,
    const vector<bool>& propagate_down,
    vector<Blob<Dtype>*>* bottom) {
  const Dtype* top_diff = top[0]->gpu_diff();
  const Dtype* top_data = top[0]->gpu_data();
  Dtype* bottom_diff = (*bottom)[0]->mutable_gpu_diff();
  const Dtype* bottom_data = (*bottom)[0]->gpu_data();
  Dtype* norm_data = norm_.mutable_gpu_data();
  Dtype* temp_dot_data = temp_dot_.mutable_gpu_data();
  Dtype* temp_data = square_.mutable_gpu_data();//just reuse the square_
  int num = top[0]->num();
  int channels = top[0]->channels();
  int spatial_dim = top[0]->height() * top[0]->width();
  caffe_copy(top[0]->count(), top_diff, bottom_diff);
  caffe_copy(top[0]->count(), bottom_data, temp_data);

  // b_diff = t_diff / norm - dot(t_diff, t_data) / (norm)^2 * bottom_data
  // temp_dot_data = dot(t_diff, t_data)
  kernel_channel_dot<Dtype><<<CAFFE_GET_BLOCKS(num * spatial_dim),
      CAFFE_CUDA_NUM_THREADS>>>(num, channels, spatial_dim, top_diff, top_data,
      temp_dot_data);
  // temp_dot_data /= (norm)^2
  caffe_gpu_div<Dtype>(num * spatial_dim, temp_dot_data, norm_data, temp_dot_data);
  caffe_gpu_div<Dtype>(num * spatial_dim, temp_dot_data, norm_data, temp_dot_data);
  // bottom_diff = top_diff, bottom_diff /= norm
  kernel_channel_div<Dtype><<<CAFFE_GET_BLOCKS(num * spatial_dim),
      CAFFE_CUDA_NUM_THREADS>>>(num, channels, spatial_dim, bottom_diff,
      norm_data);
  // temp_data = bottom_data, temp_data *= temp_dot_data
  kernel_channel_mul<Dtype><<<CAFFE_GET_BLOCKS(num * spatial_dim),
      CAFFE_CUDA_NUM_THREADS>>>(num, channels, spatial_dim, temp_data,
      temp_dot_data); 
  // bottom_diff += -temp_data
  caffe_gpu_axpy<Dtype>(top[0]->count(), Dtype(-1.0), temp_data, 
      bottom_diff);
}


INSTANTIATE_CLASS(GRNLayer);


}  // namespace caffe
```

## 6. layer test 
caffe/src/caffe/test/test\_grn\_layer.cpp

```c++
#include <cmath>
#include <cstring>
#include <vector>

#include "gtest/gtest.h"

#include "caffe/blob.hpp"
#include "caffe/common.hpp"
#include "caffe/filler.hpp"
#include "caffe/vision_layers.hpp"

#include "caffe/test/test_caffe_main.hpp"
#include "caffe/test/test_gradient_check_util.hpp"

namespace caffe {

template <typename TypeParam>
class GRNLayerTest : public MultiDeviceTest<TypeParam> {
  typedef typename TypeParam::Dtype Dtype;
 protected:
  GRNLayerTest()
      : blob_bottom_(new Blob<Dtype>(2, 10, 4, 5)),
        blob_top_(new Blob<Dtype>()) {
    // fill the values
    FillerParameter filler_param;
    GaussianFiller<Dtype> filler(filler_param);
    filler.Fill(this->blob_bottom_);
    blob_bottom_vec_.push_back(blob_bottom_);
    blob_top_vec_.push_back(blob_top_);
  }
  virtual ~GRNLayerTest() { delete blob_bottom_; delete blob_top_; }
  Blob<Dtype>* const blob_bottom_;
  Blob<Dtype>* const blob_top_;
  vector<Blob<Dtype>*> blob_bottom_vec_;
  vector<Blob<Dtype>*> blob_top_vec_;
};

TYPED_TEST_CASE(GRNLayerTest, TestDtypesAndDevices);

TYPED_TEST(GRNLayerTest, TestForward) {
  typedef typename TypeParam::Dtype Dtype;
  LayerParameter layer_param;
  GRNLayer<Dtype> layer(layer_param);
  layer.SetUp(this->blob_bottom_vec_, &(this->blob_top_vec_));
  layer.Forward(this->blob_bottom_vec_, &(this->blob_top_vec_));
  // Test sum
  for (int i = 0; i < this->blob_bottom_->num(); ++i) {
    for (int k = 0; k < this->blob_bottom_->height(); ++k) {
      for (int l = 0; l < this->blob_bottom_->width(); ++l) {
        Dtype sum = 0;
        for (int j = 0; j < this->blob_top_->channels(); ++j) {
          sum += pow(this->blob_top_->data_at(i, j, k, l), 2.0);
        }
        EXPECT_GE(sum, 0.999);
        EXPECT_LE(sum, 1.001);
        // Test exact values
        Dtype scale = 0;
        for (int j = 0; j < this->blob_bottom_->channels(); ++j) {
          scale += pow(this->blob_bottom_->data_at(i, j, k, l), 2.0);
        }
        for (int j = 0; j < this->blob_bottom_->channels(); ++j) {
          EXPECT_GE(this->blob_top_->data_at(i, j, k, l) + 1e-4,
              (this->blob_bottom_->data_at(i, j, k, l)) / sqrt(scale) )
              << "debug: " << i << " " << j;
          EXPECT_LE(this->blob_top_->data_at(i, j, k, l) - 1e-4,
              (this->blob_bottom_->data_at(i, j, k, l)) / sqrt(scale) )
              << "debug: " << i << " " << j;
        }
      }
    }
  }
}

TYPED_TEST(GRNLayerTest, TestGradient) {
  typedef typename TypeParam::Dtype Dtype;
  LayerParameter layer_param;
  GRNLayer<Dtype> layer(layer_param);
  GradientChecker<Dtype> checker(1e-2, 1e-3);
  checker.CheckGradientExhaustive(&layer, &(this->blob_bottom_vec_),
      &(this->blob_top_vec_));
}

}  // namespace caffe

```
