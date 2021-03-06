# tf-slim-Class-Activation-Map
use tensorflow slim to realize  Class Activation Map in  Googlenet or other cnn

Based on Learning Deep Features for Discriminative Localization（https://arxiv.org/pdf/1512.04150.pdf）

![class activation map](https://github.com/wpydcr/tf-slim-Class-Activation-Map/blob/master/img./6874.jpg)

![result](https://github.com/wpydcr/tf-slim-Class-Activation-Map/blob/master/img./20171220111.jpg)

We can find some packaged code of caffe model to use CAM, but hard to find some code of specifically modified net to use CAM.

This CAM(Class Activation Map) need to be changed on some net structure，I explicitly wrote how to modify it for use on any net.

This code based on Inception v3.

The most important change is to modify the part behind Convolution, which means the CAM need a Global Average Pooling and a fully connected layer.

For example:

`last_conv_net = slim.avg_pool2d(net, [8, 8], padding='VALID',
                      scope='AvgPool_1a_8x8') #1x1x2048`
                      
`logits = slim.fully_connected(net, num_classes, activation_fn=None,
                     normalizer_fn=None, scope='fc')`
                     
`w_variables = slim.get_model_variables()[-2]#10*2048`

When you finish training the net and want to generate the thermal piture, you need to use two parameter(last_conv_net & w_variables). as following:


    
      import numpy as np
      def py_returnCAMmap(last_conv_net, w_variables):
        n_feat, w, h, n= activation.shape
        act_vec = np.reshape(activation, [n_feat, w*h])
        n_top = weights_LR.shape[0]
        out = np.zeros([w, h, n_top])
        for t in range(n_top):
            weights_vec = np.reshape(weights_LR[t], [1, weights_LR[t].shape[0]])
            heatmap_vec = np.dot(weights_vec,act_vec)
            heatmap = np.reshape( np.squeeze(heatmap_vec) , [w, h])
            out[:,:,t] = heatmap
        return out
    
    
