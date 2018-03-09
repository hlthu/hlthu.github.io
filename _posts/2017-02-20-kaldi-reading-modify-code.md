---
layout: post
title: kaldi 笔记：阅读和修改代码
tags: [tkaldi, timit]
---

本文翻译和总结与 [kaldi.org](http://kaldi-asr.org/doc/tutorial_code.html)。本文主要介绍 kaldi 代码的组织结构以及依赖结构，以及一些修改和调试代码的经验。如果想更深入了解，可以点击[这里](http://kaldi-asr.org/doc/index.html)。

# 常用的使用工具

进入 `src/base/`，查看 `kaldi-common.h`，其内容主要为下：

```c++
#ifndef KALDI_BASE_KALDI_COMMON_H_
#define KALDI_BASE_KALDI_COMMON_H_ 1

#include <cstddef>
#include <cstdlib>
#include <cstring>  // C string stuff like strcpy
#include <string>
#include <sstream>
#include <stdexcept>
#include <cassert>
#include <vector>
#include <iostream>
#include <fstream>

#include "base/kaldi-utils.h"
#include "base/kaldi-error.h"
#include "base/kaldi-types.h"
#include "base/io-funcs.h"
#include "base/kaldi-math.h"

#endif  // KALDI_BASE_KALDI_COMMON_H_
```

这里主要引用了 `base/` 目录下的一些头文件，可以通过头文件的名称大致了解这些头文件的功能。包括了一些使用工具、错误处理、typedef、io 函数和数学函数等。但是这里的工具只是一套简化的实用程序，更主要的在 `util/common-utils.h` 里，其内容大致如下：

```c++
#ifndef KALDI_UTIL_COMMON_UTILS_H_
#define KALDI_UTIL_COMMON_UTILS_H_

#include "base/kaldi-common.h"
#include "util/parse-options.h"
#include "util/kaldi-io.h"
#include "util/simple-io-funcs.h"
#include "util/kaldi-holder.h"
#include "util/kaldi-table.h"
#include "util/table-types.h"
#include "util/text-utils.h"

#endif  // KALDI_UTIL_COMMON_UTILS_H_
```

可以看到该文件不仅引用了 `base/kaldi-common.h`，还引用了其他一些文件，包括参数选择、kaldi 的 io 机制等。

至于为什么把 `base` 和 `util` 这两个目录隔离开，Daniel Povey 解释道：这是因为有关矩阵的代码（在 `matrix/` 目录下）只依赖于 `base/` 目录下的内容，这样做可以减少 `matrix/` 里代码的依赖。

# 矩阵库（修改和调试代码）

现在看文件 `matrix/matrix-lib.h`，其代码内容大致如下：

```c++
#ifndef KALDI_MATRIX_MATRIX_LIB_H_
#define KALDI_MATRIX_MATRIX_LIB_H_

#include "matrix/cblas-wrappers.h"
#include "base/kaldi-common.h"
#include "matrix/kaldi-vector.h"
#include "matrix/kaldi-matrix.h"
#include "matrix/sp-matrix.h"
#include "matrix/tp-matrix.h"
#include "matrix/matrix-functions.h"
#include "matrix/srfft.h"
#include "matrix/compressed-matrix.h"
#include "matrix/sparse-matrix.h"
#include "matrix/optimization.h"

#endif
```

可以看到，在外部其只引用了 `base/kaldi-common.h`，而在内部其主要引用了包括向量、矩阵和优化等相关的头文件。这个矩阵库基本上是用 C++ 包装了 BLAS 和 LAPACK。文件 `sp-matrix.h` 和 `tp-matrix.h` 分别涉及对称矩阵和三角矩阵。快速扫一下 `kaldi-matrix.h`，会对有关矩阵的代码有一个直观的了解，它由表示矩阵的 C++ 类组成：包括 `MatrixBase` 类和 `Matrix`、`SubMatrix` 类等。关于 kaldi 的 matrix，可以看[这里](http://kaldi-asr.org/doc/matrix.html)。

此时，我们就可以修改代码并编译，我们将会在 `matrix/matrix-lib-test.cc` 文件里添加一个测试函数。如前所述，测试函数是为了检查是否有错误，如果出错返回非零值。

我们将为函数 `Vector::AddVec` 添加一个测试例程。这个函数就是给一个向量乘上一个常量，并返回另一个向量。添加的代码如下：

```c++
template<class Real>
void UnitTestAddVec() {
  // note: Real will be float or double when instantiated.
  int32 dim = 1 + Rand() % 10;
  Vector<Real> v(dim); w(dim); // two vectors the same size.
  InitRand(&v);
  InitRand(&w);
  Vector<Real> w2(w); // w2 is a copy of w.
  Real f = RandGauss();
  w.AddVec(f, v); // w <-- w + f v
  for (int32 i = 0; i < dim; i++) {
    Real a = w(i), b = f * w2(i) + v(i);
    AssertEqual(a, b); // will crash if not equal to within
    // a tolerance.
  }
}
```

注意这里 `AssertEqual()` 比较两个数是否相等，否则的话报错。这段代码有两个错误，第一个是

```c++
Vector<Real> v(dim); w(dim);
```

中间的分号错误，应该用逗号。还有一个就是

```c++
Real a = w(i), b = f * w2(i) + v(i);
```

`b` 的表达式有误，参考

```
w.AddVec(f, v); // w <-- w + f v
```

应该是

```c++
Real a = w(i), b = w2(i) + f * v(i);
```

把上面这段代码添加到下面这行代码前面

```c++
template<typename Real> static void MatrixUnitTest(bool full_test) {
```

并在函数 `MatrixUnitTest()` 里添加一行

```c++
UnitTestAddVec<Real>();
```

然后在命令行输入 `make test` 进行编译。发现会报错

```
matrix-lib-test.cc:4576:29: error: ‘w’ was not declared in this scope
   Vector<Real> v(dim); w(dim); // two vectors the same size.
```

这就是第一个错误，修改之，重新编译，编译成功。但是运行会出错：

```
ASSERTION_FAILED (AssertEqual():base/kaldi-math.h:276) : 'ApproxEqual(a, b, relative_tolerance)' 

[ Stack-Trace: ]

kaldi::MessageLogger::HandleMessage(kaldi::LogMessageEnvelope const&, char const*)
kaldi::MessageLogger::~MessageLogger()
kaldi::KaldiAssertFailure_(char const*, char const*, int, char const*)
./matrix-lib-test() [0x4233a4]
void kaldi::UnitTestAddVec<float>()
./matrix-lib-test() [0x42406e]
main
__libc_start_main
./matrix-lib-test() [0x4231b9]

Aborted
```

显然这是第二个错误，下面我们开始调试。输入

```shell
gdb ./matrix-lib-test
```
关于 gdb 的调试方法可以参考[这里](http://www.cnblogs.com/wuyuegb2312/archive/2013/03/29/2987025.html)。可以用 `r` 开始运行，`n` 是下一步。也可以用

```
(gdb) p a
$5 = -0.931363404
(gdb) p b
$6 = -0.270584524
(gdb)
```

查看变量的值。

# 声学模型

接下来看 `gmm/diag-gmm.h`，这个类定义了一个混合高斯模型（Gaussian Mixture Model），由于这个类具有很多访问接口，因此看起来可能令人困惑。查找 `private` 来查看类的成员变量：

```c++
private:
  /// Equals log(weight) - 0.5 * (log det(var) + mean*mean*inv(var))
  Vector<BaseFloat> gconsts_;
  bool valid_gconsts_;   ///< Recompute gconsts_ if false
  Vector<BaseFloat> weights_;        ///< weights (not log).
  Matrix<BaseFloat> inv_vars_;       ///< Inverted (diagonal) variances
  Matrix<BaseFloat> means_invvars_;  ///< Means times inverted variance

  // merged_components_logdet computes logdet for merged components
  // f1, f2 are first-order stats (normalized by zero-order stats)
  // s1, s2 are second-order stats (normalized by zero-order stats)
  BaseFloat merged_components_logdet(BaseFloat w1, BaseFloat w2,
                                     const VectorBase<BaseFloat> &f1,
                                     const VectorBase<BaseFloat> &f2,
                                     const VectorBase<BaseFloat> &s1,
                                     const VectorBase<BaseFloat> &s2) const;

private:
  const DiagGmm &operator=(const DiagGmm &other);  // Disallow assignment
```

这些变量通常以下划线结尾。这应该清楚我们如何存储GMM。 这只是一个GMM，而不是一个完整的GMM集合(GMMs)。

再查看 `gmm/am-diag-gmm.h` 文件，该类定义了一个 `private` 类型的变量来存储 GMMs：

```c++
private:
  std::vector<DiagGmm*> densities_;
```

注意到这个类并没有继承任何东西。一个自然的问题是：转换在哪里，决策树在哪里，HMM拓扑在哪里？ 所有这些事物与声学模型保持分离，因为研究人员可能希望替换声学似然性，同时保持系统的其余部分相同。 我们稍后会来介绍这个其他部分。

# 特征提取

现在看 `feat/feature-mfcc.h` 文件，请关注结构体 `MfccOptions`：

```c++
/// MfccOptions contains basic options for computing MFCC features.
struct MfccOptions {
  FrameExtractionOptions frame_opts;
  MelBanksOptions mel_opts;
  int32 num_ceps;  // e.g. 13: num cepstral coeffs, counting zero.
  bool use_energy;  // use energy; else C0
  BaseFloat energy_floor;
  bool raw_energy;  // If true, compute energy before preemphasis and windowing
  BaseFloat cepstral_lifter;  // Scaling factor on cepstra for HTK compatibility.
                              // if 0.0, no liftering is done.
  bool htk_compat;  // if true, put energy/C0 last and introduce a factor of
                    // sqrt(2) on C0 to be the same as HTK.

  MfccOptions() : mel_opts(23),
                  // defaults the #mel-banks to 23 for the MFCC computations.
                  // this seems to be common for 16khz-sampled data,
                  // but for 8khz-sampled data, 15 may be better.
                  num_ceps(13),
                  use_energy(true),
                  energy_floor(0.0),  // not in log scale: a small value e.g. 1.0e-10
                  raw_energy(true),
                  cepstral_lifter(22.0),
                  htk_compat(false) {}

  void Register(OptionsItf *opts) {
    frame_opts.Register(opts);
    mel_opts.Register(opts);
    opts->Register("num-ceps", &num_ceps,
                   "Number of cepstra in MFCC computation (including C0)");
    opts->Register("use-energy", &use_energy,
                   "Use energy (not C0) in MFCC computation");
    opts->Register("energy-floor", &energy_floor,
                   "Floor on energy (absolute, not relative) in MFCC computation");
    opts->Register("raw-energy", &raw_energy,
                   "If true, compute energy before preemphasis and windowing");
    opts->Register("cepstral-lifter", &cepstral_lifter,
                   "Constant that controls scaling of MFCCs");
    opts->Register("htk-compat", &htk_compat,
                   "If true, put energy or C0 last and use a factor of sqrt(2) on "
                   "C0.  Warning: not sufficient to get HTK compatible features "
                   "(need to change other parameters).");
  }
};
```

注意到，它有一个成员是其本身。再看看 Register 函数，这是Kaldi选项类的标准。

接下来看 `featbin/compute-mfcc-feats.cc`，这是一个命令行程序，在其中搜索 Register 函数

```c++
    ParseOptions po(usage);
    MfccOptions mfcc_opts;
    ···········
    // Register the MFCC option struct
    mfcc_opts.Register(&po);
```

为了查看 MFCC 特征提取时的参数列表，请直接运行 `./compute-mfcc-feats`。

```
Usage:  compute-mfcc-feats [options...] <wav-rspecifier> <feats-wspecifier>

Options:
  --blackman-coeff            : Constant coefficient for generalized Blackman window. (float, default = 0.42)
  --cepstral-lifter           : Constant that controls scaling of MFCCs (float, default = 22)
  --channel                   : Channel to extract (-1 -> expect mono, 0 -> left, 1 -> right) (int, default = -1)
  --debug-mel                 : Print out debugging information for mel bin computation (bool, default = false)
  --dither                    : Dithering constant (0.0 means no dither) (float, default = 1)
  --energy-floor              : Floor on energy (absolute, not relative) in MFCC computation (float, default = 0)
  --frame-length              : Frame length in milliseconds (float, default = 25)
  --frame-shift               : Frame shift in milliseconds (float, default = 10)
  --high-freq                 : High cutoff frequency for mel bins (if < 0, offset from Nyquist) (float, default = 0)
  --htk-compat                : If true, put energy or C0 last and use a factor of sqrt(2) on C0.  Warning: not sufficient to get HTK compatible features (need to change other parameters). (bool, default = false)
  --low-freq                  : Low cutoff frequency for mel bins (float, default = 20)
  --min-duration              : Minimum duration of segments to process (in seconds). (float, default = 0)
  --num-ceps                  : Number of cepstra in MFCC computation (including C0) (int, default = 13)
  --num-mel-bins              : Number of triangular mel-frequency bins (int, default = 23)
  --output-format             : Format of the output files [kaldi, htk] (string, default = "kaldi")
  --preemphasis-coefficient   : Coefficient for use in signal preemphasis (float, default = 0.97)
  --raw-energy                : If true, compute energy before preemphasis and windowing (bool, default = true)
  --remove-dc-offset          : Subtract mean from waveform on each frame (bool, default = true)
  --round-to-power-of-two     : If true, round window size to power of two. (bool, default = true)
  --sample-frequency          : Waveform data sample frequency (must match the waveform file, if specified there) (float, default = 16000)
  --snip-edges                : If true, end effects will be handled by outputting only frames that completely fit in the file, and the number of frames depends on the frame-length.  If false, the number of frames depends only on the frame-shift, and we reflect the data at the ends. (bool, default = true)
  --subtract-mean             : Subtract mean of each feature file [CMS]; not recommended to do it this way.  (bool, default = false)
  --use-energy                : Use energy (not C0) in MFCC computation (bool, default = true)
  --utt2spk                   : Utterance to speaker-id map rspecifier (if doing VTLN and you have warps per speaker) (string, default = "")
  --vtln-high                 : High inflection point in piecewise linear VTLN warping function (if negative, offset from high-mel-freq (float, default = -500)
  --vtln-low                  : Low inflection point in piecewise linear VTLN warping function (float, default = 100)
  --vtln-map                  : Map from utterance or speaker-id to vtln warp factor (rspecifier) (string, default = "")
  --vtln-warp                 : Vtln warp factor (only applicable if vtln-map not specified) (float, default = 1)
  --window-type               : Type of window ("hamming"|"hanning"|"povey"|"rectangular"|"blackmann") (string, default = "povey")

Standard options:
  --config                    : Configuration file to read (this option may be repeated) (string, default = "")
  --help                      : Print out usage message (bool, default = false)
  --print-args                : Print the command line arguments (to stderr) (bool, default = true)
  --verbose                   : Verbose level (higher->more logging) (int, default = 0)
```

下面是计算 MFCC 特征的一个例子，请用自己的数据目录和特征目录代替 `/dev/null`。

```
$ featbin/compute-mfcc-feats --raw-energy=false ark:/dev/null ark:/dev/null
# 示例输出
featbin/compute-mfcc-feats --raw-energy=false ark:/dev/null ark:/dev/null 
LOG (compute-mfcc-feats:main():compute-mfcc-feats.cc:181)  Done 0 out of 0 utterances.
```

# 声学决策树

接下来查看 `tree/build-tree.h` 文件，请查找到 `BuildTree`函数。

```c++
/**
 *  BuildTree is the normal way to build a set of decision trees.
 *  The sets "phone_sets" dictate how we set up the roots of the decision trees.
 *  each set of phones phone_sets[i] has shared decision-tree roots, and if
 *  the corresponding variable share_roots[i] is true, the root will be shared
 *  for the different HMM-positions in the phone.  All phones in "phone_sets"
 *  should be in the stats (use FixUnseenPhones to ensure this).
 *  if for any i, do_split[i] is false, we will not do any tree splitting for
 *  phones in that set.
 * @param qopts [in] Questions options class, contains questions for each key
 *                   (e.g. each phone position)
 * @param phone_sets [in] Each element of phone_sets is a set of phones whose
 *                 roots are shared together (prior to decision-tree splitting).
 * @param phone2num_pdf_classes [in] A map from phones to the number of
 *                 \ref pdf_class "pdf-classes"
 *                 in the phone (this info is derived from the HmmTopology object)
 * @param share_roots [in] A vector the same size as phone_sets; says for each
 *                phone set whether the root should be shared among all the
 *                pdf-classes or not.
 * @param do_split [in] A vector the same size as phone_sets; says for each
 *                phone set whether decision-tree splitting should be done
 *                 (generally true for non-silence phones).
 * @param stats [in] The statistics used in tree-building.
 * @param thresh [in] Threshold used in decision-tree splitting (e.g. 1000),
 *                   or you may use 0 in which case max_leaves becomes the
 *                    constraint.
 * @param max_leaves [in] Maximum number of leaves it will create; set this
 *                  to a large number if you want to just specify  "thresh".
 * @param cluster_thresh [in] Threshold for clustering leaves after decision-tree
 *                  splitting (only within each phone-set); leaves will be combined
 *                  if log-likelihood change is less than this.  A value about equal
 *                  to "thresh" is suitable
 *                  if thresh != 0; otherwise, zero will mean no clustering is done,
 *                  or a negative value (e.g. -1) sets it to the smallest likelihood
 *                  change seen during the splitting algorithm; this typically causes
 *                  about a 20% reduction in the number of leaves.
 
 * @param P [in] The central position of the phone context window, e.g. 1 for a
 *                triphone system.
 * @return  Returns a pointer to an EventMap object that is the tree.
*/

EventMap *BuildTree(Questions &qopts,
                    const std::vector<std::vector<int32> > &phone_sets,
                    const std::vector<int32> &phone2num_pdf_classes,
                    const std::vector<bool> &share_roots,
                    const std::vector<bool> &do_split,
                    const BuildTreeStatsType &stats,
                    BaseFloat thresh,
                    int32 max_leaves,
                    BaseFloat cluster_thresh,  // typically == thresh.  If negative, use smallest split.
                    int32 P);
```

这是构建决策树的最高层函数，注意到其返回一个指向 `EventMap` 变量的指针。这是一种存储函数的类型，这个函数的形式是 `(key,value)` 对，该函数在 `tree/event-map.h` 中定义。其中 `key` 和 `value` 都是整数，但 `key` 表示语音语境位置（通常为0,1或2），`value` 表示音素。也有一个特殊的 `key`，为 -1，其大致表示 HMM 中的位置。

BuildTree 函数的主要输入是 BuildTreeStatsType 类型，它是一个typedef，如下所示：

```c++
typedef vector<pair<EventType, Clusterable*> > BuildTreeStatsType;
```

而 EvenType 是如下的 typedef:

```c++
typedef vector<pair<EventKeyType, EventValueType> > EventType;
```

其中 EvenType 代表的是一组 `(key,value)` 对。比如一个典型的例子是 `{ {-1, 1}, {0, 15}, {1, 21}, {2, 38} }`，意味着音素 21 的左边上下文是音素 0，右边上下文是音素 38，而 pdf 类是 1。Clusterable *指针是一个指向虚拟类的指针，它具有一个通用接口，支持将统计信息添加在一起并评估某种目标函数。在通常情况下，它实际指向一个包括足够用于评估对角高斯概率密度函数的统计量的类。

进入一个实验目录，例如 `egs/timit/s5`，我们将查看决策树是怎么建立的。执行

```
$ cat exp/tri1/log/acc_tree.1.log
# 示例输出：
# acc-tree-stats --ci-phones=1 exp/mono_ali/final.mdl "ark,s,cs:apply-cmvn  --utt2spk=ark:data/train/split8/1/utt2spk scp:data/train/split8/1/cmvn.scp scp:data/train/split8/1/feats.scp ark:- | add-deltas  ark:- ark:- |" "ark:gunzip -c exp/mono_ali/ali.1.gz|" exp/tri1/1.treeacc 
# Started at Fri Feb 17 21:09:14 CST 2017
#
acc-tree-stats --ci-phones=1 exp/mono_ali/final.mdl 'ark,s,cs:apply-cmvn  --utt2spk=ark:data/train/split8/1/utt2spk scp:data/train/split8/1/cmvn.scp scp:data/train/split8/1/feats.scp ark:- | add-deltas  ark:- ark:- |' 'ark:gunzip -c exp/mono_ali/ali.1.gz|' exp/tri1/1.treeacc
add-deltas ark:- ark:-
apply-cmvn --utt2spk=ark:data/train/split8/1/utt2spk scp:data/train/split8/1/cmvn.scp scp:data/train/split8/1/feats.scp ark:-
LOG (apply-cmvn[5.0.51~1-cd97]:main():apply-cmvn.cc:146) Applied cepstral mean normalization to 464 utterances, errors on 0
LOG (acc-tree-stats[5.0.51~1-cd97]:main():acc-tree-stats.cc:118) Accumulated stats for 464 files, 0 failed due to no alignment, 0 failed for other reasons.
LOG (acc-tree-stats[5.0.51~1-cd97]:main():acc-tree-stats.cc:121) Number of separate stats (context-dependent states) is 18972
# Accounting: time=3 threads=1
# Ended (code 0) at Fri Feb 17 21:09:17 CST 2017, elapsed time 3 seconds
```

输出并没有十分重要的信息，但是你可以看到一个命令，即

```
acc-tree-stats --ci-phones=1 exp/mono_ali/final.mdl "ark,s,cs:apply-cmvn  --utt2spk=ark:data/train/split8/1/utt2spk scp:data/train/split8/1/cmvn.scp scp:data/train/split8/1/feats.scp ark:- | add-deltas  ark:- ark:- |" "ark:gunzip -c exp/mono_ali/ali.1.gz|" exp/tri1/1.treeacc 
```

这个程序为每一个可见的 3 音素上下文的 HMM 状态（实际上是一个概率密度函数）做一个单高斯统计。`–ci-phones` 选项可以避免为特殊音素的上下文累积单独的统计，如沉默，我们不想然它成为上下文相关的（这只是一个优化，没有这个选项也可以工作）。这个命令的输出可以被认为是上面讨论的类型 `BuildTreeStatsType`，虽然为了读它，我们必须知道它是什么具体的类型。


# HMM 拓扑

下面查看 `hmm/hmm-topology.h` 文件，类 HmmTopology 为一系列音素定义了一个 HMM 拓扑集。一般一个每一个音素可以有不同的拓扑，拓扑包括用于初始化的“默认”转换。查看头文件开头扩展注释中的示例拓扑。

```c++
/*
 // The following would be the text form for the "normal" HMM topology.
 // Note that the first state is the start state, and the final state,
 // which must have no output transitions and must be nonemitting, has
 // an exit probability of one (no other state can have nonzero exit
 // probability; you can treat the transition probability to the final
 // state as an exit probability).
 // Note also that it's valid to omit the "<PdfClass>" entry of the <State>, which
 // will mean we won't have a pdf on that state [non-emitting state].  This is equivalent
 // to setting the <PdfClass> to -1.  We do this normally just for the final state.
 // The Topology object can have multiple <TopologyEntry> blocks.
 // This is useful if there are multiple types of topology in the system.

 <Topology>
 <TopologyEntry>
 <ForPhones> 1 2 3 4 5 6 7 8 </ForPhones>
 <State> 0 <PdfClass> 0
 <Transition> 0 0.5
 <Transition> 1 0.5
 </State>
 <State> 1 <PdfClass> 1
 <Transition> 1 0.5
 <Transition> 2 0.5
 </State>
 <State> 2 <PdfClass> 2
 <Transition> 2 0.5
 <Transition> 3 0.5
 <Final> 0.5
 </State>
 <State> 3
 </State>
 </TopologyEntry>
 </Topology>
*/
```

其中有一个标签 `<PdfClass>`，其总是与HMM状态（`<State>`）相同; 但是一般来说，它并不一定得是。



